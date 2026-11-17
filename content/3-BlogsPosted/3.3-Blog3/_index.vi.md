---
title: "Blog 3 - Tìm kiếm Ngữ nghĩa với Bedrock Titan Embeddings: Tối ưu 256 chiều vector"
date: 2026-10-31
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

{{% notice info %}}
* **Tác giả:** Đỗ Trần Minh Phúc (MSSV: 23120156 — Trường ĐH Khoa học Tự nhiên, ĐHQG-HCM)
* **Chương trình:** First Cloud AI Journey (FCAJ) 2026 — AWS Vietnam
* **Dự án liên quan:** EduMedia AI — Hệ thống Lưu trữ & Tra cứu Bài giảng Thông minh với AWS Serverless & GenAI
* **Kênh công bố:** [Cộng đồng AWS Study Group (Facebook)](https://www.facebook.com/groups/awsstudygroupfcj)
{{% /notice %}}

# Tìm kiếm Ngữ nghĩa Đa phương thức với Amazon Bedrock: Tại sao 256 chiều vector là "điểm ngọt" cho Serverless?

Trong quá trình xây dựng hệ thống **EduMedia AI** phục vụ việc lưu trữ và tra cứu bài giảng cho sinh viên (ảnh chụp bảng viết tay, slide thuyết trình, file ghi âm giọng nói), bài toán lớn nhất mình phải giải quyết là: **Làm sao để sinh viên tìm kiếm bằng câu hỏi ngôn ngữ tự nhiên tiếng Việt mà hệ thống vẫn trả về đúng tài liệu liên quan?**

Nếu chỉ sử dụng tìm kiếm từ khóa truyền thống (Lexical Search), hệ thống sẽ gặp bế tắc hoàn toàn:
- Sinh viên tìm *"công thức đạo hàm cấp hai"* nhưng ảnh bảng viết tay chỉ chứa ký hiệu toán học `$f''(x)$` hoặc `$y''$`.
- Sinh viên tìm *"quy luật chuyển hóa từ lượng thành chất"* nhưng file ghi âm bài giảng Triết học của giảng viên chỉ nói về *"tích lũy về lượng dẫn đến sự thay đổi về chất"*.

Để giải quyết bài toán này, giải pháp hiển nhiên trong kỷ nguyên GenAI là sử dụng **Vector Embeddings** thông qua mô hình nền tảng **Amazon Bedrock Titan Text Embeddings V2**. Tuy nhiên, khi đưa vector vào kiến trúc Serverless, một "cái bẫy kiến trúc" chí mạng về kích thước vector bắt đầu xuất hiện.

---

### 1. Cái bẫy 1536 chiều vector trong kiến trúc Serverless NoSQL

Các mô hình nhúng văn bản phổ biến thường mặc định xuất ra vector có số chiều rất lớn: **1024 hoặc 1536 chiều**. Mỗi chiều là một số thực dấu phẩy động 32-bit (float32 = 4 bytes).

Hãy làm một phép tính nhanh về mặt chi phí và tài nguyên lưu trữ:
- **1 vector 1536 chiều:** $1536 \times 4 \approx 6.144 \text{ bytes} \approx 6.1 \text{ KB}$ dữ liệu thô.
- Khi lưu vào DynamoDB dưới dạng mảng số (Number Set hoặc List), dung lượng bị đội lên khoảng **10 – 12 KB** do overhead định dạng JSON/DynamoDB Attribute.
- Một bài giảng âm thanh dài 45 phút sau khi bóc băng qua Amazon Transcribe được chia thành **30 – 50 đoạn (chunks)** để tìm kiếm chính xác từng mốc thời gian.
- Tổng dung lượng vector chỉ riêng cho 1 bài giảng: $40 \text{ chunks} \times 12 \text{ KB} \approx 480 \text{ KB}$!

**Hậu quả kỹ thuật:**
1. **Đánh sập giới hạn cứng của DynamoDB:** Amazon DynamoDB giới hạn tối đa **400 KB** cho mỗi item. Việc lưu toàn bộ vector độ phân giải cao sẽ lập tức ném lỗi `ValidationException: Item size has exceeded the maximum allowed size`.
2. **Chi phí RCU/WCU tăng phi mã:** DynamoDB tính 1 WCU cho mỗi 1 KB ghi. Một file tốn hàng chục WCU chỉ để ghi mảng số, làm phình to hóa đơn hàng tháng.
3. **Độ trễ tính toán Cosine Distance trên Lambda:** Khi tính độ tương đồng vector (Cosine Similarity) giữa câu query của người dùng và hàng nghìn chunks trong bộ nhớ Lambda bằng Python thuần, ma trận 1536 chiều khiến thời gian phản hồi API bị kéo dài từ 20ms lên đến hơn 180ms.

---

### 2. Giải pháp: Tận dụng cơ chế Sub-vector 256 chiều của Titan Embeddings V2

Rất may mắn, mô hình **Amazon Titan Text Embeddings V2 (`amazon.titan-embed-text-v2:0`)** được AWS tích hợp sẵn tính năng giảm chiều không gian nhúng mà vẫn giữ được tính toàn vẹn ngữ nghĩa cao. API cho phép chỉ định trực tiếp tham số:
`dimensions: 256 | 512 | 1024`

```python
# Cấu hình gọi Amazon Bedrock Titan Text Embeddings V2 với 256 dimensions
response = bedrock_runtime.invoke_model(
    modelId="amazon.titan-embed-text-v2:0",
    body=json.dumps({
        "inputText": chunk_text,
        "dimensions": 256,
        "normalize": True
    })
)
embedding_256 = json.loads(response["body"].read())["embedding"]
```

#### Đánh giá thực nghiệm giữa 1024 chiều vs 256 chiều trên tập dữ liệu bài giảng:
| Tiêu chí | 1024 chiều (Mặc định) | 256 chiều (Tối ưu) | Mức độ cải thiện |
| :--- | :---: | :---: | :---: |
| **Dung lượng raw vector** | ~4.1 KB | **~1.0 KB** | **Giảm 75%** |
| **Dung lượng DynamoDB item (40 chunks)** | ~360 KB (Cận giới hạn) | **~85 KB** | **Tiết kiệm 76% dung lượng** |
| **Chi phí WCU/RCU trên DynamoDB** | 4 – 5 WCU/chunk | **1 WCU/chunk** | **Giảm 75% cước DB** |
| **Thời gian tính Cosine trên Lambda** | ~145 ms | **~24 ms** | **Nhanh hơn 6 lần** |
| **Độ chính xác Top-5 Semantic Recall** | 94.2% | **92.6%** | *Chỉ giảm 1.6% (không đáng kể)* |

Nhờ tham số `normalize: True`, các vector trả về đã được chuẩn hóa độ dài về 1 (Unit vector). Khi tính Cosine Similarity, chúng ta chỉ cần thực hiện phép nhân vô hướng (Dot Product) đơn giản:
$$\text{Similarity} = \mathbf{A} \cdot \mathbf{B} = \sum_{i=1}^{256} A_i B_i$$
Phép tính 256 phép nhân và cộng số thực chạy cực nhanh trên runtime Python của AWS Lambda mà không cần phải cài thêm các thư viện C nặng nề như NumPy hay SciPy (giúp giữ dung lượng Lambda Deployment Package siêu nhẹ dưới 10MB).

---

### 3. Thiết kế phòng vệ: Graceful Degradation khi Bedrock chạm Quota

Trong môi trường thực tế, đặc biệt là với các tài khoản dùng AWS Promotional Credits hoặc tài khoản mới, Amazon Bedrock áp đặt giới hạn On-Demand Token Quota rất nghiêm ngặt (có thể gặp `ThrottlingException: Too many tokens per day` hoặc giới hạn Requests-Per-Minute).

Nếu không thiết kế phòng vệ, khi Bedrock bị throttle, toàn bộ tính năng tìm kiếm của web sẽ sập với mã lỗi `HTTP 500 Internal Server Error`.

Trong **EduMedia AI**, mình xây dựng cơ chế **Phòng vệ suy giảm nhẹ nhàng (Graceful Degradation)**:

```python
def execute_search_plan(query_text, user_filters):
    try:
        # Bước 1: Thử dùng Bedrock Nova Lite để trích xuất Planner & Titan 256-d Embeddings
        search_plan = generate_bedrock_plan(query_text)
        vector = generate_titan_embedding(query_text, dimensions=256)
        return perform_hybrid_search(search_plan, vector)
    except (ThrottlingException, ClientError) as e:
        logger.warning(f"Bedrock Quota Exceeded. Activating Lexical Fallback: {e}")
        # Bước 2: Tự động kích hoạt cơ chế Fallback Heuristic
        return fallback_lexical_search(query_text, user_filters)
```

Khi Bedrock không khả dụng:
1. Hệ thống tự động bắt lỗi và ghi nhận custom metric `BedrockThrottledFallback` lên Amazon CloudWatch.
2. Ngay lập tức chuyển hướng câu query sang **Lexical Keyword Match**: quét trên trường `labels` (do Rekognition trích xuất) và `transcript_text` (do Transcribe trích xuất) đã được đánh chỉ mục trong DynamoDB.
3. Người dùng trên giao diện Web vẫn nhận được danh sách bài giảng phù hợp mà không hề gặp màn hình trắng hay thông báo lỗi hệ thống.

---

### 4. Đúc kết bài học kiến trúc (Key Architectural Takeaways)

1. **Hiểu bản chất bài toán trước khi chạy theo công nghệ:** Không phải cứ vector càng nhiều chiều thì tìm kiếm càng tốt. Với các ứng dụng tra cứu tài liệu học tập vừa và nhỏ, việc giảm từ 1024 xuống 256 chiều tiết kiệm được hơn 75% chi phí lưu trữ và tăng tốc độ xử lý gấp nhiều lần mà trải nghiệm tìm kiếm của sinh viên không hề bị suy giảm.
2. **Luôn thiết kế cơ chế Fallback cho các dịch vụ AI bên ngoài:** AI/LLM là dịch vụ phụ thuộc vào quota và độ trễ mạng. Một kiến trúc Serverless chuẩn **AWS Well-Architected** bắt buộc phải có cơ chế suy thoái mềm để hệ thống luôn phục vụ được người dùng cuối trong mọi tình huống.
3. **Giữ Lambda tinh gọn:** Chọn vector 256 chiều chuẩn hóa giúp thuật toán so khớp chạy mượt mà bằng Python thuần, không phụ thuộc thư viện bên ngoài, giữ cold start của Lambda luôn dưới 300ms.
