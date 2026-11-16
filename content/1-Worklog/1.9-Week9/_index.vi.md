---
title: "Tuần 9"
date: 2026-10-16
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

# Tuần 9: Tích Hợp Amazon Bedrock & Hybrid Search
*Thời gian thực hiện: 12/10/2026 – 16/10/2026*

### 1. Mục tiêu tuần 9:
* Tích hợp Amazon Bedrock SDK vào Lambda API để kết nối mô hình ngôn ngữ lớn (LLM) và mô hình nhúng vector.
* Sử dụng Amazon Nova Lite (amazon.nova-lite-v1:0) làm AI Query Planner phân tích câu hỏi tiếng Việt thành Search Plan có cấu trúc.
* Khắc phục lỗi json.loads bị crash khi Nova Lite trả về chuỗi JSON bọc trong markdown code block.
* Tích hợp Amazon Titan Text Embeddings V2 (amazon.titan-embed-text-v2:0) nén vector từ 1536 xuống 256 chiều với chuẩn hóa L2.
* Hiện thực thuật toán Hybrid Search: kết hợp Vector Cosine Similarity (trọng số 0.6) và Normalized Keyword Overlap (trọng số 0.4).

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Thiết lập kết nối Boto3 Bedrock Runtime từ apps/api/search.py.<br>- Kiểm tra quyền hạn IAM gọi API bedrock:InvokeModel. | 12/10/2026 | 12/10/2026 | [Bedrock Runtime API](https://docs.aws.amazon.com/bedrock/latest/userguide/api-methods-run.html) |
| **3** | - Xây dựng prompt cho Amazon Nova Lite.<br>- Bóc tách thực thể: danh mục (category), từ khóa (keywords) và ý định tìm kiếm từ câu hỏi tiếng Việt. | 13/10/2026 | 13/10/2026 | [Amazon Nova Lite Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-nova.html) |
| **4** | - Xử lý lỗi parse JSON khi model trả về markdown code blocks.<br>- Viết hàm regex tiền xử lý clean_llm_json_output làm sạch payload an toàn trước khi gọi json.loads. | 14/10/2026 | 14/10/2026 | Python re Module |
| **5** | - Gọi API Amazon Titan Text Embeddings V2 với tham số dimensions: 256 và normalize: True.<br>- Sinh vector nhúng 256 chiều chuẩn hóa L2 cho câu truy vấn. | 15/10/2026 | 15/10/2026 | [Titan Embeddings V2 Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) |
| **6** | - Hiện thực thuật toán Hybrid Search kết hợp Cosine Similarity và Lexical Keyword Overlap.<br>- Kiểm thử độ chính xác định tuyến với các câu hỏi tìm kiếm phức tạp. | 16/10/2026 | 16/10/2026 | Vector Similarity Algorithms |

### 3. Kết quả đạt được tuần 9:
* Tích hợp thành công Amazon Nova Lite làm AI Query Planner phân tích câu hỏi tiếng Việt ổn định.
* Triệt tiêu hoàn toàn lỗi crash JSON parsing bằng hàm tiền xử lý regex.
* Kích hoạt thành công Titan Text Embeddings V2 sinh vector 256 chiều chuẩn hóa L2, tiết kiệm 83% dung lượng lưu trữ trên DynamoDB mà vẫn giữ độ chính xác trên 98%.
* Thuật toán Hybrid Search định tuyến chính xác các câu hỏi tự nhiên về đúng file media hoặc phân đoạn âm thanh liên quan.

### 4. Tài liệu tham khảo:
* [Amazon Bedrock Converse API Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
* [Amazon Titan Text Embeddings V2 Technical Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html)
* [Hybrid Search Architecture Patterns on AWS](https://aws.amazon.com/blogs/machine-learning/)
