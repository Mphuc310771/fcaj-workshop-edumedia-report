---
title: "Kiến trúc đánh đổi & Hướng phát triển"
date: 2026-10-21
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# 5.7. Thảo luận: Kiến trúc Đánh đổi & Lộ trình Nâng cấp Enterprise (Architecture Trade-offs & Enterprise Roadmap)

Trong giai đoạn xây dựng PoC (Proof of Concept) phục vụ chương trình First Cloud AI Journey (FCAJ), mục tiêu cốt lõi của AgentSys là hiện thực hóa một hệ thống tìm kiếm đa phương tiện thông minh bằng AI, tuân thủ kiến trúc Serverless với tiêu chí **tối ưu chi phí tối đa (Zero-Idle-Cost)** và **triển khai tinh gọn (Lean Architecture)**. 

Để đạt được sự cân bằng giữa chi phí học tập/thực nghiệm và tính năng, một số quyết định đánh đổi kiến trúc (architectural trade-offs) có chủ đích đã được áp dụng. Dưới đây là phân tích phản biện kỹ thuật về 4 giới hạn hiện tại của hệ thống và giải pháp mở rộng chuẩn AWS Enterprise khi đưa vào môi trường sản xuất quy mô lớn.

---

### 1. Giới hạn In-Memory Vector Search trên RAM Lambda
* **Hiện trạng PoC:** 
  DynamoDB là cơ sở dữ liệu NoSQL dạng tài liệu/khóa-giá trị, không hỗ trợ tính toán vector similarity (ANN). Để thực hiện Hybrid Search, AgentSys áp dụng kỹ thuật Two-Stage Search: Bedrock đóng vai trò Query Planner trích xuất category để lọc nhanh qua DynamoDB GSI (category-index), sau đó kéo một nhóm ứng viên (Candidate Pool giới hạn tối đa 100 bản ghi mới nhất) về bộ nhớ RAM của Lambda API để tính Cosine Similarity bằng Python.
* **Đánh giá hạn chế:** 
  Giải pháp này hoàn toàn miễn phí duy trì cơ sở hạ tầng (không tốn chi phí cụm vector database chạy liên tục). Tuy nhiên, khi quy mô catalog vượt quá hàng chục nghìn bản ghi, việc giới hạn 100 ứng viên sẽ gây ra hiện tượng bỏ sót dữ liệu (recall drop) đối với các file cũ hơn. Nếu quét toàn bộ bảng (Full Table Scan), chi phí RCU của DynamoDB và thời gian thực thi của Lambda sẽ tăng tuyến tính `O(N)`.
* **Giải pháp Enterprise chuẩn AWS:**
  - Tích hợp **Amazon OpenSearch Serverless (Vector Engine)** hoặc **Amazon Aurora Serverless (PostgreSQL với extension pgvector)**.
  - Các vector 256-d/1024-d sẽ được lưu trữ và lập chỉ mục bằng thuật toán HNSW (Hierarchical Navigable Small World) hoặc IVF. Khi người dùng truy vấn, phép tính k-NN diễn ra trực tiếp ở tầng lưu trữ với độ trễ dưới 20ms mà không cần tải dữ liệu thô vào RAM của hàm tính toán Lambda.

---

### 2. Kỹ thuật nén chiều Vector & Nâng cấp Titan Embeddings V2
* **Hiện trạng PoC:** 
  Trong các phiên bản đầu, việc giảm chiều vector từ 1536-d xuống 256-d được thực hiện thông qua việc lấy mẫu bước nhảy (uniform striding) nhằm tiết kiệm không gian lưu trữ và giới hạn kích thước document trên DynamoDB (tối đa 400KB/item).
* **Đánh giá hạn chế:** 
  Các chiều trong không gian tiềm ẩn (latent space) của mạng nơ-ron sâu không có tính độc lập tuyến tính. Việc trích xuất thô bạo có thể làm méo mó cấu trúc tô-pô ngữ nghĩa (semantic manifold), ảnh hưởng đến độ phân định của điểm Cosine Similarity.
* **Giải pháp Enterprise chuẩn AWS:**
  - Hệ thống đã được nâng cấp hỗ trợ **Amazon Titan Text Embeddings V2 (amazon.titan-embed-text-v2:0)**. Mô hình V2 hỗ trợ cơ chế nén chiều tự nhiên (Matryoshka Representation Learning) cho phép chỉ định trực tiếp dimensions: 256 kèm theo chuẩn hóa L2 (normalize: true) ngay từ API Bedrock.
  - Trong tương lai, việc kết hợp **Amazon Titan Multimodal Embeddings** sẽ cho phép sinh vector trực tiếp từ hình ảnh và video frame, hợp nhất vector hình ảnh và văn bản vào cùng một không gian đa chiều duy nhất.

---

### 3. Mô hình dữ liệu DynamoDB và Xử lý phân mảnh (Single-Table Design)
* **Hiện trạng PoC:** 
  Bảng AgentSysMediaMetadata được cấu hình với Partition Key đơn (file\_id). Khi mở rộng tính năng băm nhỏ bài giảng âm thanh dài thành các đoạn nhỏ (chunking), các chunk được lưu trữ với khóa file\_id = f"{file\_id}#CHUNK#{index}". Do thiếu Sort Key chuyên biệt, hàm gom chunk (get\_file\_chunks) buộc phải áp dụng biểu thức lọc (FilterExpression).
* **Đánh giá hạn chế:** 
  Việc sử dụng FilterExpression trên bảng không có Sort Key tiêu tốn dung lượng đọc (RCU) vì DynamoDB vẫn phải đọc toàn bộ khối dữ liệu trước khi lọc bỏ.
* **Giải pháp Enterprise chuẩn AWS:**
  - Tái cấu trúc bảng theo mô hình **DynamoDB Single-Table Design chuẩn hóa**:
    - **Partition Key (PK):** FILE#&lt;file\_id&gt;
    - **Sort Key (SK):** METADATA (cho bản ghi file chính) hoặc CHUNK#&lt;index&gt; (cho từng phân đoạn bài giảng).
    Query(PK="FILE#123", SK begins\_with "CHUNK#"). Thao tác này đạt độ phức tạp `O(k)`, truy xuất chính xác các chunk cần thiết mà không phải scan bất kỳ bản ghi thừa nào.

---

### 4. Điều phối Map-Reduce cho bài giảng âm thanh lớn
* **Hiện trạng PoC:** 
  Khi Amazon Transcribe hoàn tất, sự kiện EventBridge kích hoạt worker Lambda xử lý tóm tắt và gom cụm tuần tự qua các chunk, đồng thời kích hoạt luồng DynamoDB Stream để kiểm chứng tính bất biến và chống race condition bằng ConditionExpression.
* **Đánh giá hạn chế:** 
  Nếu file âm thanh là bài giảng dài từ 60 đến 120 phút (tạo ra 40–80 chunks), việc gọi mô hình Bedrock tuần tự trong một hàm Lambda duy nhất có thể chạm ngưỡng giới hạn thời gian chạy (Lambda Timeout 120s – 900s).
* **Giải pháp Enterprise chuẩn AWS:**
  - Thay thế vòng lặp xử lý nội bộ bằng **AWS Step Functions với Distributed Map State**:
    1. **State 1 (Split):** Lambda chia nhỏ transcript thành 50 chunks và lưu vào S3/DynamoDB.
    2. **State 2 (Distributed Map):** Step Functions tự động phân tán (fan-out) đồng thời 50 Lambda workers chạy song song, mỗi worker gọi Bedrock xử lý độc lập cho 1 chunk trong 2 giây.
    3. **State 3 (Reduce):** Tự động gom (fan-in) kết quả từ 50 chunks và gọi hàm Aggregator Lambda tổng hợp bản tóm tắt hoàn chỉnh.
  - Mô hình này giảm thời gian xử lý toàn bộ bài giảng 2 tiếng từ **5 phút xuống còn dưới 10 giây**, đồng thời loại bỏ triệt để nguy cơ Lambda timeout.

---

### 5. Cơ chế S3-Offloading chống tràn 400KB DynamoDB & Tối ưu Container Hiệu năng cao
* **Hiện trạng & Kỹ thuật đã giải quyết:**
  - **S3-Offloading (>250KB):** DynamoDB giới hạn cứng 400KB cho mỗi document. Đối với các file ghi âm bài giảng dài hoặc video thuyết trình, văn bản bóc băng (transcript) thô có thể vượt quá 400KB–1MB. Để triệt tiêu rủi ro lỗi ValidationException: Item size has exceeded 400KB, AgentSys tự động nhận diện transcript vượt ngưỡng 250KB và chuyển toàn văn bản sang S3 (transcripts/raw/{file\_id}.json). Bảng DynamoDB chỉ lưu trích đoạn xem trước (15.000 ký tự đầu) cùng URI định danh transcript\_s3\_uri và cờ has\_extended\_transcript: true. Frontend React hỗ trợ tải xuống toàn văn (Raw JSON) và hiển thị modal xem trực tiếp.
  - **Tái sử dụng Lambda Warm Container:** Giữ nguyên connection pool và TLS handshake của Boto3 giữa các lần kích hoạt Lambda, loại bỏ lệnh reset runtime ở đầu handler, giúp giảm 50–120ms độ trễ cho mỗi message SQS hoặc request API.
  - **Bộ đệm truy vấn LRU (_PLAN_CACHE):** Lưu trữ trong RAM Lambda tối đa 256 kế hoạch truy vấn đã phân tích từ Bedrock trong 1 giờ. Khi chịu tải 50 requests đồng thời, bộ đệm mang lại hệ số tăng tốc **3.74x** (từ ~3,194ms xuống ~855ms) và tiết kiệm 100% chi phí token Bedrock cho các câu hỏi lặp lại.
* **Giải pháp Enterprise chuẩn AWS:**
  - Kết hợp **Amazon ElastiCache Serverless (Redis)** làm distributed cache dùng chung cho toàn bộ dàn Lambda containers thay vì in-memory cache cục bộ.

---

### 6. Cẩm nang Phản biện Kỹ thuật & Tối ưu Chi phí Chuyên sâu (Defense Q&A Guide)

Nhằm chuẩn bị vững vàng cho buổi nghiệm thu dự án thực tập trước Hội đồng chuyên môn, dưới đây là 3 câu hỏi kiểm toán chuyên sâu về kiến trúc thực tế và câu trả lời phân tích định lượng chuẩn mực:

#### ❓ Câu hỏi 1: "Khi trích xuất 1 keyframe mỗi 30 giây cho 250 phút video bằng ffmpeg, số ảnh phát sinh thêm này đã được tính vào chi phí Rekognition và Bedrock ở Kịch bản 2 chưa?"
* **Trả lời chuẩn mực:** 
  Toàn bộ chi phí này đã được dự phòng đầy đủ trong biên độ dao động của Kịch bản 2. Cụ thể: 250 phút video với tần suất 1 frame / 30 giây sinh ra đúng **500 keyframes**. Khi gửi 500 frames này qua Amazon Rekognition (DetectLabels + DetectText) và Amazon Bedrock (Titan Multimodal Embeddings G1), chi phí phát sinh là:
  $$\text{Chi phí} = (500 \times 2 \times \$0.001) + (500 \times \$0.00006) = \$1.03\text{ USD/tháng}$$
  Khoản phát sinh **$1.03 USD** này nằm gọn hoàn toàn trong khoảng đệm an toàn **$4.50 USD** (giữa mức nền **$9.77** và mức trần **$14.27 USD**) mà nhóm đã thiết lập trong mô hình FinOps.

#### ❓ Câu hỏi 2: "Tại sao nhóm dùng LLM (Amazon Nova Micro) để xử lý câu truy vấn Tiếng Việt mà không dùng dịch vụ Amazon Translate?"
* **Trả lời chuẩn mực:** 
  Việc lựa chọn Amazon Nova Micro thay vì Amazon Translate xuất phát từ 2 lợi thế vượt trội về FinOps và Kiến trúc:
  1. **Hiệu quả kinh tế (Tiết kiệm gấp ~1.500 lần):** Amazon Translate có giá niêm yết là **$15.00 USD / 1 triệu ký tự**. Trong khi đó, `Amazon Nova Micro` trên Bedrock có giá chỉ **$0.035 USD / 1 triệu input tokens** (tương đương ~3.5 – 4 triệu ký tự). Do đó, sử dụng Nova Micro giúp tiết kiệm hơn 1.500 lần chi phí dịch thuật.
  2. **Mô hình xử lý tích hợp (Single-Pass Multi-Tasking):** Trong duy nhất 1 lệnh gọi prompt tới Nova Micro, mô hình vừa thực hiện phân tích ý định người dùng (Intent Routing), vừa trích xuất bộ lọc thời gian/môn học (Metadata Filtering), vừa chuyển ngữ câu mô tả sang Tiếng Anh chuẩn thị giác cho Titan Embeddings. Điều này loại bỏ hoàn toàn độ trễ mạng (network latency) của việc gọi nối tiếp hai dịch vụ riêng biệt.

#### ❓ Câu hỏi 3: "Hàm Lambda dùng ffmpeg cắt frame từ video lớn (hàng trăm MB). Bộ nhớ tạm /tmp mặc định chỉ có 512MB, liệu có nguy cơ tràn đĩa hoặc phát sinh phí nâng cấp /tmp không?"
* **Trả lời chuẩn mực:** 
  Hệ thống áp dụng kỹ thuật **Streaming Frame Extraction qua S3 Presigned URL**: Thay vì tải toàn bộ file video `.mp4` về ổ đĩa tạm `/tmp`, Lambda truyền trực tiếp đường dẫn S3 Presigned URL vào câu lệnh ffmpeg:
  ```bash
  ffmpeg -ss <timestamp> -i "https://<bucket>.s3.amazonaws.com/<key>?<presigned_params>" -frames:v 1 -f image2 pipe:1
  ```
  Nhờ giao thức **HTTP Byte-Range Requests** được hỗ trợ nguyên bản trên Amazon S3, `ffmpeg` chỉ đọc phần tiêu đề metadata (moov atom) rồi nhảy trực tiếp tới đúng byte dữ liệu tại giây cần lấy và xuất frame ảnh tĩnh thẳng vào bộ nhớ RAM. Nhờ đó, Lambda không tiêu tốn dung lượng `/tmp`, hoàn toàn miễn nhiễm với lỗi tràn đĩa (`No space left on device`) và hoàn thành tác vụ chỉ trong 1–2 giây.

---

### Bảng so sánh tổng hợp: Kiến trúc PoC vs Kiến trúc Enterprise Production

| Tiêu chí kỹ thuật | Kiến trúc PoC hiện tại (EduMedia AI) | Kiến trúc Enterprise AWS đề xuất |
| :--- | :--- | :--- |
| **Cơ chế Vector Search** | Candidate Pool (100 items) + Cosine RAM Lambda | Amazon OpenSearch Serverless / pgvector (HNSW Index) |
| **Không gian Embeddings** | Titan V2 (256-d) + Fallback Titan V1 Striding | Titan Multimodal Embeddings V2 Native (256-d / 1024-d) |
| **Cấu trúc dữ liệu** | Single PK (file\_id) + FilterExpression | Composite Primary Key (PK: FILE#id, SK: CHUNK#idx) |
| **Bảo vệ giới hạn 400KB** | Tự động S3-Offloading cho transcript > 250KB | S3 Intelligent-Tiering + OpenSearch Document Pointer |
| **Xử lý Map-Reduce** | EventBridge Worker + DynamoDB Streams Demo | AWS Step Functions (Distributed Map State) |
| **Tối ưu hóa Vòng đời Container** | Warm Container Boto3 reuse + LRU \_PLAN\_CACHE | Provisioned Concurrency + ElastiCache Serverless |
| **Độ trễ xử lý âm thanh** | Tuyến tính `O(N)` theo số chunks | Song song `O(1)` phân tán đồng thời |
| **Chi phí hạ tầng nhàn rỗi** | **0.00 USD / tháng** (Zero-Idle Serverless) | ~20 – 50 USD / tháng (OpenSearch Serverless OCU tối thiểu) |
| **Mục tiêu tối ưu** | Tiết kiệm chi phí thực nghiệm, dễ debug, bảo vệ PoC | Độ tin cậy cao, mở rộng hàng triệu files, SLA doanh nghiệp |
