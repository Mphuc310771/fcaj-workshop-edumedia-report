---
title: "Tuần 6"
date: 2026-09-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Tuần 6: Khởi Tạo Hạ Tầng S3 & Pipeline Tiếp Nhận Media
*Thời gian thực hiện: 21/09/2026 – 25/09/2026*

### 1. Mục tiêu tuần 6:
* Triển khai mã nguồn hàm Lambda xử lý tiếp nhận media (apps/workers/ingestion.py) bằng Python 3.12.
* Tối ưu hóa hiệu năng Lambda thông qua kỹ thuật tái sử dụng Boto3 connection pool và TLS handshake ngoài handler.
* Xây dựng bộ định tuyến (router) phân loại định dạng media an toàn dựa trên MIME type và phần mở rộng tệp.
* Đóng gói và triển khai stack hạ tầng lên môi trường AWS thông qua sam build và sam deploy.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Khởi tạo cấu trúc thư mục dự án edumedia-ai/.<br>- Viết module apps/workers/ingestion.py tiếp nhận thông điệp từ hàng đợi SQS. | 21/09/2026 | 21/09/2026 | [AWS Lambda Programming Model](https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model.html) |
| **3** | - Tối ưu hóa container warm-up.<br>- Khởi tạo các Boto3 clients (S3, Rekognition, Transcribe) bên ngoài hàm handler để tái sử dụng connection pool. | 22/09/2026 | 22/09/2026 | [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) |
| **4** | - Viết hàm phân loại media detect_media_type.<br>- Phân biệt chính xác giữa ảnh (JPG, PNG, WEBP), âm thanh (MP3, WAV, M4A) và video (MP4, MOV). | 23/09/2026 | 23/09/2026 | Python mimetypes Module |
| **5** | - Viết bộ unit test đầu tiên kiểm thử logic router định dạng file bằng pytest.<br>- Kiểm tra các tình huống tệp không có phần mở rộng hoặc sai định dạng. | 24/09/2026 | 24/09/2026 | [Pytest Documentation](https://docs.pytest.org/) |
| **6** | - Chạy sam build và thực hiện deployment stack lên tài khoản AWS tại us-east-1.<br>- Kiểm tra log luồng SQS sang Lambda trên CloudWatch xác nhận nhận file thành công. | 25/09/2026 | 25/09/2026 | [SAM CLI Deploy](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html) |

### 3. Kết quả đạt được tuần 6:
* Xây dựng hoàn chỉnh mô-đun tiếp nhận dữ liệu apps/workers/ingestion.py chạy trên runtime Python 3.12.
* Tối ưu hóa hiệu năng container: khai báo Boto3 client toàn cục giúp tái sử dụng kết nối TCP và TLS session, giảm 50–120ms độ trễ trên mỗi lần kích hoạt hàm warm.
* Bộ định tuyến phân loại media xử lý chính xác cả 3 nhóm định dạng (Ảnh, Âm thanh, Video), ngăn ngừa việc gửi nhầm định dạng tới các dịch vụ AI.
* Viết thành công bộ unit tests cơ bản cho router, đảm bảo tệp lạ hoặc không rõ nguồn gốc được cách ly an toàn.
* Stack hạ tầng SAM agentsys-stack được deploy thành công lên AWS us-east-1; kiểm tra CloudWatch Logs xác nhận luồng sự kiện từ S3 qua SQS vào Lambda chạy thông suốt.

### 4. Tài liệu tham khảo:
* [Optimizing AWS Lambda performance](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
* [Building Serverless Applications with SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started.html)
* [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
