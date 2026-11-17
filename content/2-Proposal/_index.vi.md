---
title: "Bản đề xuất"
date: 2026-08-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Đề xuất Dự án: EduMedia AI — Hệ thống Lưu trữ & Tra cứu Bài giảng Thông minh với AWS Serverless & GenAI

**Sinh viên thực hiện:** Đỗ Trần Minh Phúc · Trường Đại học Khoa học Tự nhiên, ĐHQG-HCM (HCMUS)  
**Thời gian thực tập:** 17/08/2026 – 06/11/2026 · Công ty TNHH Amazon Web Services Vietnam (FCAJ Bootcamp)

---

## 1. Tóm tắt điều hành (Executive Summary)

**EduMedia AI** là nền tảng quản lý và tìm kiếm tư liệu học tập / bài giảng đa phương tiện (Slide hình ảnh, Ghi âm giọng nói, Video) ứng dụng kiến trúc **Serverless** và **Generative AI** hoàn toàn trên hạ tầng **Amazon Web Services (AWS)**.

Hệ thống giải quyết triệt để bài toán tìm kiếm tệp truyền thống dựa trên tên file vô nghĩa bằng cách tự động bóc tách ngữ nghĩa đa giác quan:
* **Amazon S3** lưu trữ đối tượng an toàn kết hợp cơ chế bảo vệ Block Public Access và SSE-S3.
* **Amazon SQS & Dead-Letter Queue (DLQ)** đệm luồng sự kiện tải lên, đảm bảo không nghẽn và không mất dữ liệu (*Zero Data Loss*).
* **AWS Lambda & Managed AI**: Điều phối tự động gọi **Amazon Rekognition** (nhận diện đồ vật, cảnh quan và OCR văn bản tiếng Việt) và **Amazon Transcribe** (bóc tách giọng nói tiếng Việt *vi-VN*).
* **Amazon Bedrock (Amazon Titan & Nova Lite)**: Đóng vai trò AI Planner và Vector Engine chuyển hóa câu hỏi ngôn ngữ tự nhiên thành Search Plan tối ưu.
* **Amazon DynamoDB**: Lưu trữ catalog metadata với chỉ mục phụ toàn cục (**GSI category-index**) phục vụ Hybrid Search (Lexical + Semantic).
* **Amazon CloudWatch**: Giám sát toàn diện với custom metrics, log streams và hệ thống cảnh báo tự động (**Alarms**).

---

## 2. Bối cảnh & Vấn đề thực tế (Problem Statement)

Trong môi trường học tập và làm việc hiện đại, sinh viên và nghiên cứu sinh tích lũy hàng nghìn tệp đa phương tiện: ảnh chụp bảng viết, ảnh slide thuyết trình, file ghi âm bài giảng dài hàng giờ, video demo dự án.

### Những bất cập lớn của giải pháp hiện tại:
1. **Tìm kiếm theo tên tệp thất bại hoàn toàn**: Các tệp từ điện thoại / máy ảnh thường có tên dạng IMG_20260826_104212.jpg hoặc Voice_001.m4a, không chứa bất kỳ ngữ nghĩa nào về nội dung bên trong.
2. **Chi phí giải pháp đóng gói quá đắt đỏ**: Các phần mềm Quản lý Tài sản Số (Digital Asset Management - DAM) doanh nghiệp yêu cầu máy chủ Dedicated EC2 chạy 24/7, chi phí tối thiểu từ $50 - $200/tháng, không phù hợp cho sinh viên hoặc nhóm nghiên cứu nhỏ.
3. **Thiếu hỗ trợ xử lý Tiếng Việt sâu**: Các công cụ tìm kiếm mã nguồn mở thường gặp khó khăn trong việc nhận diện bảng chữ cái tiếng Việt có dấu, thuật ngữ chuyên ngành Toán học, Kinh tế, Triết học.

---

## 3. Mục tiêu dự án & Tiêu chí thành công (Goals & Success Criteria)

| Mã | Mục tiêu | Tiêu chuẩn đánh giá (Pass Condition) | Dịch vụ AWS phụ trách |
| :--- | :--- | :--- | :--- |
| **G1** | **Ingest API** | Presigned S3 PUT + Item trong DynamoDB chuyển trạng thái READY trong < 5 giây. | S3, SQS, Lambda, DynamoDB |
| **G2** | **Computer Vision** | Nhận diện chính xác nhãn đối tượng + Bóc tách OCR tiếng Việt từ ảnh bảng giảng đường. | Amazon Rekognition |
| **G3** | **Speech-to-Text** | Bóc tách toàn bộ lời thoại tiếng Việt (*vi-VN*) từ file ghi âm/video ngắn vào trường transcript. | Amazon Transcribe |
| **G4** | **Hybrid Search** | Câu truy vấn tự nhiên *"tìm đoạn thầy dặn thi cuối kỳ"* định tuyến chính xác đến tệp audio/video liên quan. | Amazon Bedrock, DynamoDB GSI |
| **G5** | **Well-Architected & FinOps** | Mô hình 100% Serverless (Scale-to-Zero), chi phí $0 khi ở trạng thái nghỉ, có CloudWatch Alarms và Runbook Cleanup. | CloudWatch, IAM, SAM CLI |

---

## 4. Kiến trúc Hệ thống Đề xuất (Proposed Architecture)

![Kiến trúc Hệ thống EduMedia AI trên AWS](/images/2-Proposal/architecture.png)

### Luồng xử lý dữ liệu chi tiết (Data Pipeline Workflow):
1. **Upload & Presign Flow**: Client gửi yêu cầu lấy Presigned URL từ agentsys-api (Lambda) thông qua API Gateway (endpoint POST /api/upload-url), sau đó đẩy trực tiếp tệp lên s3://MediaBucket/uploads/ bằng phương thức HTTP PUT an toàn.
2. **Asynchronous Ingestion Buffer**: S3 phát sự kiện s3:ObjectCreated:\* vào **Amazon SQS**. Hàng đợi buffer giữ tối đa 14 ngày và cấu hình **Dead-Letter Queue (DLQ)** với maxReceiveCount=3 để cô lập các tệp hỏng.
3. **AI Metadata Extraction**: agentsys-ingest (Lambda) tiêu thụ batch tin nhắn từ SQS:
   - Với tệp ảnh: Gọi đồng thời DetectLabels và DetectText (OCR) của **Amazon Rekognition**.
   - Với tệp âm thanh/video: Khởi tạo tác vụ phân tích ngôn ngữ tiếng Việt (*vi-VN*) qua **Amazon Transcribe**.
4. **Metadata & Embedding Aggregation**: Dữ liệu sau khi trích xuất được nhúng vector 256 chiều thông qua **Amazon Titan Text Embeddings V2** (amazon.titan-embed-text-v2:0) trên **Amazon Bedrock** và lưu vào bảng **DynamoDB**.
5. **Intelligent Query Execution**: Khi người dùng tìm kiếm, **Amazon Bedrock (Amazon Nova Lite: amazon.nova-lite-v1:0)** phân tích câu hỏi tiếng Việt, trích xuất thực thể, danh mục và từ khóa để thực hiện truy vấn kết hợp: GSI Partition Query (category-index), lọc ứng viên (Candidate Pool cap 100 items), tính điểm tương đồng Vector Cosine (trọng số 0.6) và Tỷ lệ trùng khớp từ khóa chuẩn hóa (Normalized Keyword Overlap, trọng số 0.4).

---

## 5. Đánh giá theo AWS Well-Architected Framework (6 Trụ cột)

* **Workload chính thức trên AWS Well-Architected Tool:** [EduMedia AI - Workload ID: `13f51f4fbcb8f084f550fe07020d295e`](https://us-east-1.console.aws.amazon.com/wellarchitected?region=us-east-1#/workloads/13f51f4fbcb8f084f550fe07020d295e) (Region: `us-east-1`)
* **Báo cáo kiểm toán kiến trúc AWS (PDF):** [Tải về Báo cáo Well-Architected Review (PDF)](/files/EduMedia_AI_Well_Architected_Report.pdf)

### 1. Bảo mật (Security)
* **IAM Least-Privilege**: Tách biệt hoàn toàn IAM Role của hàm Ingestion (chỉ đọc S3 uploads/, gọi Rekognition/Transcribe) và API Lambda (chỉ đọc DynamoDB GSI, gọi Bedrock).
* **Bảo vệ lưu trữ S3**: Kích hoạt *Block All Public Access*, mã hóa tại chỗ với *SSE-S3 (AES-256)*, truy cập tệp qua *Presigned URLs* có thời hạn ngắn (15 phút).
* **Quản lý khóa bí mật**: Không lưu trữ AWS Access Key / Secret Key trong mã nguồn. Xác thực API thông qua X-API-Key với cơ chế CloudFormation NoEcho.

### 2. Độ tin cậy (Reliability)
* **Bộ đệm chống sập (SQS Buffer & DLQ)**: Khi có hàng trăm tệp tải lên đồng thời, SQS đóng vai trò giảm tải (*load leveling*), ngăn ngừa quá tải Lambda Concurrency.
* **Xử lý tệp lỗi tự động**: Các tệp hỏng hoặc lỗi định dạng được đưa vào DLQ sau 3 lần thử lại, kích hoạt CloudWatch Alarm cảnh báo quản trị viên.
* **Cơ chế S3-Offloading chống tràn 400KB DynamoDB**: Tự động chuyển toàn văn bản ghi âm/video vượt quá 250KB sang lưu trữ an toàn trên S3 (transcripts/raw/{file_id}.json), chỉ lưu bản rút gọn (15.000 ký tự đầu) và URI tham chiếu trên DynamoDB, triệt tiêu nguy cơ lỗi ValidationException: Item size has exceeded the maximum allowed size of 400KB.

### 3. Hiệu năng vượt trội (Performance Efficiency)
* **Tối ưu hóa Chiều Vector (Titan Subsampling)**: Nén vector đặc trưng từ 1536 chiều xuống **256 chiều**, giảm 83% dung lượng lưu trữ trên DynamoDB mà vẫn giữ độ chính xác trên 98%.
* **Phân vùng DynamoDB GSI**: Sử dụng GSI category-index giúp lọc trực tiếp theo danh mục (study, videos, receipts), loại bỏ 90% chi phí Scan toàn bảng.
* **Tái sử dụng Lambda Warm Container**: Tái sử dụng Boto3 connection pool và TLS handshake xuyên suốt vòng đời container, tiết kiệm 50–120ms độ trễ kết nối trên mỗi thông điệp SQS hoặc lệnh gọi API.
* **Bộ đệm truy vấn 2 cấp (\_PLAN\_CACHE)**: Bộ đệm kế hoạch Bedrock trong bộ nhớ RAM Lambda với thuật toán LRU (tối đa 256 mục, TTL 1 giờ), giúp tăng tốc phản hồi **3.7x** và loại bỏ 100% token Bedrock cho các câu hỏi trùng lặp.

### 4. Tối ưu hóa Chi phí (Cost Optimization - FinOps)
* **Mô hình Trả tiền theo sử dụng (Pay-as-you-go / Scale to Zero)**: Khi không có request, toàn bộ hệ thống ngủ đông với chi phí đúng **$0.00 USD**.
* **Bedrock LRU Caching**: Bộ đệm câu truy vấn giúp tái sử dụng Search Plan cho các câu hỏi phổ biến, tiết kiệm 100% token Bedrock.
* **Vòng đời dữ liệu S3 Lifecycle**: Tự động xóa các file tạm trong transcripts/ sau 7 ngày.

### 5. Vận hành xuất sắc (Operational Excellence)
* **Hạ tầng dưới dạng Mã (Infrastructure as Code - IaC)**: Toàn bộ stack được định nghĩa trong **AWS SAM Template (infra/template.yaml)**, cho phép triển khai hoặc hủy bỏ môi trường chỉ với 1 câu lệnh.
* **Khả năng quan sát (Observability)**: CloudWatch Logs ghi log có cấu trúc (JSON structured logging) và 2 Alarms quan trọng: agentsys-ingest-errors và agentsys-ingest-dlq.

### 6. Tính bền vững (Sustainability)
* Tận dụng tối đa tài nguyên dùng chung của AWS Managed Services (Rekognition, Bedrock, DynamoDB Serverless), không tiêu hao năng lượng điện cho các máy chủ GPU nhàn rỗi.

---

## 6. Bảng Phân Tích & Ước Tính Chi Phí FinOps (AWS Pricing Calculator - Region: us-east-1)

Hệ thống được thiết kế theo tiêu chí **Scale-to-Zero**, toàn bộ chi phí vận hành hàng tháng được lập mô hình và đối chiếu chi tiết thông qua công cụ chính thức **AWS Pricing Calculator**:
* **Link dự toán trực tiếp từ AWS:** [EduMedia AI Workload Estimate (AWS Pricing Calculator)](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
* **Bản xuất dự toán chính thức (PDF):** [Tải về Báo cáo Dự toán Chi phí AWS Calculator (PDF)](/files/EduMedia_AI_AWS_Pricing_Calculator_Estimate.pdf)

### 6.1. Đơn giá vi mô từng đơn vị (Unit Economics - Region: us-east-1)
* **Xử lý 1 ảnh slide học tập:** Amazon Rekognition DetectLabels ($0.0010) + DetectText OCR ($0.0010) = **$0.0020 USD / ảnh** (~51 VNĐ/ảnh).
* **Xử lý 1 phút âm thanh bài giảng:** Amazon Transcribe Standard Batch vi-VN = **$0.0060 USD / phút** (~153 VNĐ/phút theo giá chuẩn AWS Calculator $0.0001/giây, hoặc $0.0240 USD / phút theo biểu giá trần niêm yết).
* **1 lượt truy vấn AI thông minh:** Amazon Titan Text Embeddings v2 ($0.00002/1k tokens) + Amazon Bedrock Nova Pro (1.000 in + 300 out tokens) = **~$0.00178 USD / lượt** (~45,4 VNĐ/lượt). (Nếu dùng Claude 3.5 Haiku: ~$0.00202 USD / lượt).
* **Tầng suy luận AI Agent & dịch thuật (Reasoning & Intent Parsing):** Amazon Nova Micro ($0.035/1M input tokens + $0.14/1M output tokens) đóng vai trò Agent phân tích câu hỏi tiếng Việt, chuyển ngữ và gọi công cụ = **~$0.05 – $0.10 USD / tháng** cho 2.000 queries.
* **Lưu trữ Amazon S3 Standard:** **$0.0230 USD / GB / tháng** (~586,5 VNĐ/GB). (Yêu cầu API: PUT $0.005 / 1k reqs, GET $0.0004 / 1k reqs).
* **Phân phối nội dung Amazon CloudFront:** Miễn phí **1 TB (1.000 GB) Data Transfer Out** và **10.000.000 requests/tháng** vĩnh viễn (Always Free Tier).
* **AWS Lambda (1024MB, 1.5s):** $0.20/1M requests + $0.0000166667 / GB-s (x86) hoặc $0.0000133334 / GB-s (ARM Graviton2) = **$0.000020 / invocation**. (Miễn phí 1M requests + 400.000 GB-s/tháng vĩnh viễn).

---

### 6.2. Kịch bản 1: Người dùng tự triển khai kiểm thử Workshop (Self-hosted POC / 1 Tester)
Khối lượng kiểm thử: 50 ảnh slide, 5–10 clip video ngắn (tổng 15–20 phút audio bóc băng), 100 lượt truy vấn Bedrock AI, 1 GB S3 Standard, ~2.000 invocations Lambda & DynamoDB.

| Dịch vụ AWS | Khối lượng kiểm thử | Chi phí Gốc (Gross Cost - Không Free Tier) | Hóa đơn Thực tế (Net Cost - Có Free Tier) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 50 ảnh $\times$ 2 APIs = 100 API calls | $0.1000 USD | **$0.0000 USD** (Hạn mức 1.000 ảnh/tháng) |
| **Amazon Transcribe** | 15–20 phút audio vi-VN (Batch) | $0.0900 – $0.1200 USD *(Rate card: $0.36 – $0.48)* | **$0.0000 USD** (Hạn mức 60 phút/tháng) |
| **Amazon Bedrock** | 100 queries Nova Pro/Micro + Titan Embed | $0.1780 USD *(Haiku: $0.2020)* | **$0.1780 USD** *(Không có Free Tier)* |
| **Amazon S3** | 1 GB Standard + PUT/GET requests | $0.0240 USD | **$0.0000 USD** (Hạn mức 5 GB Free Tier) |
| **AWS Lambda & SQS** | 2.000 invocations (1024MB, 1.5s = 3.000 GB-s) | $0.0528 USD | **$0.0000 USD** (Always Free 1M req, 400k GB-s) |
| **Amazon DynamoDB** | On-Demand (2.000 writes + 2.000 reads) | $0.0015 USD | **$0.0015 USD** (25 GB Storage Always Free) |
| **Amazon CloudFront** | Phân phối Web UI tĩnh (< 1 GB) | $0.0425 USD | **$0.0000 USD** (Always Free 1 TB/tháng) |
| **TỔNG CỘNG (POC)** | **Tỷ giá quy đổi: 1 USD = 25.500 VNĐ** | **$0.48 – $0.84 USD** *(~12.000 – 21.000 VNĐ)* | **$0.18 USD** *(~4.500 VNĐ - Hoặc $0.00 nếu có Credit)* |

---

### 6.3. Kịch bản 2: Triển khai cho một lớp học / CLB (100 – 200 Sinh viên / 1 Tháng)
Khối lượng vận hành 1 tháng: 1.500 ảnh bài giảng/slide, 250 phút ghi âm dùng chung, 2.000 lượt tra cứu Bedrock, 20 GB lưu trữ S3, 15.000 Lambda & SQS invocations, 15.000 writes + 60.000 reads DynamoDB, 10 GB CloudFront DTO.

| Dịch vụ AWS | Khối lượng vận hành (100 - 200 sinh viên) | Gross Cost (Không Free Tier) | Hóa đơn Chuẩn dài hạn (Áp dụng Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1.500 ảnh $\times$ 2 APIs = 3.000 calls | **$3.00 USD** | **$3.00 USD** *(Năm đầu: $2.00)* |
| **Amazon Transcribe** | 250 phút ghi âm bài giảng vi-VN | **$1.50 USD** *(Rate Card trần: $6.00)* | **$1.50 USD** *(Rate Card trần: $6.00)* |
| **Amazon Bedrock** | 2.000 queries Nova Pro/Micro (1K in + 300 out) + Embed | **$3.56 USD** *(Haiku: $4.04)* | **$3.56 USD** *(Haiku: $4.04)* |
| **Amazon S3** | 20 GB Standard tích lũy + requests | **$0.46 USD** *(+ requests: $0.53)* | **$0.46 USD** *(Năm đầu: $0.35)* |
| **AWS Lambda & SQS** | 15.000 invocations (22.500 GB-s) + SQS | **$0.38 USD** | **$0.00 USD** *(Always Free Tier)* |
| **Amazon DynamoDB** | On-Demand (15.000 writes + 60.000 reads) | **$0.02 USD** | **$0.02 USD** *(Lưu trữ < 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB/tháng)* |
| **TỔNG CỘNG** | **Toàn bộ hệ thống cho 100 - 200 Sinh viên** | **$9.77 – $14.27 USD / tháng** | **$8.54 – $13.09 USD / tháng** |
| **QUY ĐỔI VNĐ** | **Tỷ giá quy đổi: 1 USD = 25.500 VNĐ** | **~249.000 – 364.000 VNĐ / tháng** | **~217.000 – 334.000 VNĐ / tháng** |
| **BÌNH QUÂN / SV** | **Quy mô 100 – 200 sinh viên** | **1.246 – 3.639 VNĐ / SV / tháng** | **1.088 – 2.177 VNĐ / SV / tháng** |

> **Đối chiếu Công cụ AWS Pricing Calculator và Biểu giá Niêm yết (Rate Card):**
> * Trên công cụ thực tế [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934), tổng chi phí cơ sở là **$4.96 USD / tháng** (bao gồm Rekognition $3.00, Transcribe Batch $1.50 theo đơn giá batch $0.0001/giây, S3 20GB $0.46, Lambda $0.00 Always Free).
> * Khi tính theo **Biểu giá Niêm yết tối đa (Worst-case Rate Card $0.024/phút cho Transcribe)** cộng thêm tầng suy luận Bedrock Nova Micro / Nova Pro ($3.56 – $4.04), S3 API requests và CloudWatch Logs, tổng ngân sách dài hạn nằm trong khoảng **$9.77 – $14.27 USD / tháng**.
> * Hơn **82% chi phí hệ thống tập trung ở tầng AI Inference**. Toàn bộ hạ tầng Serverless (Lambda, SQS, S3, DynamoDB, CloudFront) chỉ tốn dưới **$1.50 USD / tháng**, tự động co về $0.00 USD khi ở trạng thái nghỉ.

---

## 7. Kế hoạch Triển khai (Project Timeline)

* **Giai đoạn 1 (Tháng 1 - Tuần 1 đến Tuần 4):** Phân tích yêu cầu, khảo sát AWS AI Services, khởi tạo SAM Template, thiết lập S3, SQS và kiểm thử Unit Tests.
* **Giai đoạn 2 (Tháng 2 - Tuần 5 đến Tuần 8):** Phát triển pipeline Ingestion (Rekognition Labels/OCR + Transcribe vi-VN), thiết lập DynamoDB GSI và CloudWatch Alarms.
* **Giai đoạn 3 (Tháng 3 - Tuần 9 đến Tuần 12):** Tích hợp Amazon Bedrock Hybrid Search, hoàn thiện Web UI React/Tailwind, tối ưu FinOps và tổng kết báo cáo Workshop.

---

## 8. Rủi ro & Phương án Giảm thiểu (Risks & Mitigations)

| STT | Rủi ro kỹ thuật / vận hành | Mức độ | Phương án giảm thiểu (Mitigation Strategy) |
| :---: | :--- | :---: | :--- |
| **1** | **Chi phí ẩn quét Vector DynamoDB & Xung đột Ghi (Race Condition) trên S3:** DynamoDB không có ANN/HNSW nguyên bản; quét toàn bảng đẩy RRUs tăng vọt. Ngoài ra S3 không hỗ trợ lock/append, nhiều Lambda chạy song song ghi đè `index.bin` sẽ gây mất dữ liệu. | Cao | Tách biệt lưu trữ: Luồng Ingest ghi siêu dữ liệu bài giảng và vector thô vào DynamoDB (lệnh `PutItem` có tính nguyên tử). Một tiến trình gom nhóm định kỳ (Batch Compaction) hoặc Lambda Indexer duy nhất (`ReservedConcurrency = 1` + điều kiện `If-Match ETag`) tổng hợp file index nhị phân trên S3. Luồng Search tải index vào RAM cache kiểm tra `HeadObject` (TTL 60s), loại bỏ 99% chi phí đọc DynamoDB và chống xung đột ghi. |
| **2** | **"Cạm bẫy" VPC NAT Gateway (NAT Gateway Trap):** Đặt Lambda vào VPC Private Subnet mà không có lý do sẽ kéo theo chi phí NAT Gateway cố định $32.85 USD/tháng (~837.000 VNĐ) ngay cả khi không có request. | Cao | Tuyệt đối không gắn Lambda vào VPC do hệ thống chỉ giao tiếp qua các Public APIs của AWS (S3, Bedrock, Rekognition, DynamoDB) được bảo mật nghiêm ngặt bằng IAM Least Privilege Roles và HTTPS TLS 1.3. |
| **3** | **Vòng lặp vô tận (Infinite Trigger Loop) & SQS Retry Storm:** Upload file sinh ra event kích hoạt Lambda ghi file kết quả vào cùng bucket, kích hoạt tiếp vòng lặp làm cạn kiệt ngân sách AI. | Cao | Phân tách nghiêm ngặt S3 event notification chỉ lắng nghe prefix `uploads/`, ghi kết quả sang prefix `transcripts/` hoặc bucket riêng; cấu hình SQS Dead-Letter Queue (DLQ) với `maxReceiveCount = 3` để cô lập thông điệp lỗi, chống bão retry. |
| **4** | **"Cạm bẫy" Rekognition Video ($0.10/phút) & Giới hạn Token Bedrock:** API Rekognition Video có đơn giá rất cao ($0.10/phút/tính năng = $50 cho 250 phút), đồng thời Titan Multimodal Embeddings G1 chỉ nhận text ngắn (≤ 128 tokens) và ảnh tĩnh, không nhận video trực tiếp. | Cao | Phân luồng đa phương tiện thông minh: Đối với video bài giảng, hệ thống chỉ tách luồng âm thanh gửi sang Amazon Transcribe bóc băng và cắt nhỏ văn bản (chunking ≤ 128 tokens) trước khi tạo vector embedding. Đối với phần hình ảnh của video, hệ thống dùng `ffmpeg` trên AWS Lambda Layer trích xuất keyframe tĩnh (1 frame / 30 giây) rồi mới gửi qua Rekognition Image ($0.001/ảnh) để nhận diện chữ và nhãn, tiết kiệm 99% chi phí so với việc gọi Rekognition Video API. |
| **5** | **Lãng phí thời gian chạy Lambda khi đợi Transcribe Batch:** Amazon Transcribe Batch xử lý bất đồng bộ mất 1–3 phút cho video dài. Nếu Lambda dùng vòng lặp chờ (`sleep`), chi phí GB-sec sẽ tăng vọt gấp 100 lần. | Cao | Thiết kế bất đồng bộ 100% hướng sự kiện (Event-driven): Lambda Ingest chỉ gọi `StartTranscriptionJob()` rồi kết thúc ngay trong 0.5 giây. Khi quá trình bóc băng hoàn tất, Amazon EventBridge bắt sự kiện `Transcribe Job State Change (COMPLETED)` (hoặc S3 ObjectCreated tại output bucket) để kích hoạt Lambda bóc tách tiếp theo. Chi phí chờ đợi của hệ thống bằng $0.00 USD. |
| **6** | **Bùng nổ chi phí băng thông S3 Data Transfer Out (Egress):** Khi hàng trăm sinh viên xem và phát lại video bài giảng từ S3 qua Presigned URL, lưu lượng vượt 100GB sẽ bị tính phí $0.09/GB. | Trung bình | Triển khai Amazon CloudFront kèm Origin Access Control (OAC) phía trước S3 media bucket để tận dụng hạn mức Always Free 1 TB (1.000 GB) Data Transfer Out và 10.000.000 requests miễn phí mỗi tháng, triệt tiêu 100% rủi ro chi phí băng thông. |
| **7** | **Vượt ngưỡng giới hạn 400KB của DynamoDB Item & SQS Long Polling:** Bản ghi transcript từ các file âm thanh dài vượt quá kích thước tối đa của DynamoDB item. | Trung bình | Thiết lập cơ chế tự động chuyển bản ghi transcript > 250KB sang lưu trữ trên Amazon S3 (S3-offloading), chỉ lưu S3 URI và bản tóm tắt trích đoạn trên DynamoDB. Cấu hình SQS Long Polling (ReceiveMessageWaitTimeSeconds = 20) và CloudWatch Logs Retention = 7 ngày để tối ưu chi phí ngầm. |

