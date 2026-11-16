---
title: "Tuần 3"
date: 2026-09-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Tuần 3: Xử Lý Bất Đồng Bộ Với SQS & AWS Lambda
*Thời gian thực hiện: 31/08/2026 – 04/09/2026*

### 1. Mục tiêu tuần 3:
* Xây dựng nguyên mẫu hàm Lambda bằng Python 3.12 để tiếp nhận sự kiện tải tệp lên từ S3.
* Đánh giá rủi ro cạn kiệt Concurrency của tài khoản khi S3 kích hoạt Lambda trực tiếp (Push-based model).
* Tích hợp Amazon SQS làm bộ đệm điều tiết tải (Load Leveling) với MaximumConcurrency và ReportBatchItemFailures.
* Cấu hình Dead-Letter Queue (DLQ) và khắc phục lỗi lệch thời gian chờ (Visibility Timeout mismatch) giữa SQS và Lambda.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Viết nguyên mẫu hàm Lambda bằng Python 3.12 nhận sự kiện s3:ObjectCreated:*.<br>- Kiểm tra log ghi nhận payload trên CloudWatch Logs. | 31/08/2026 | 31/08/2026 | [AWS Lambda with Python](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) |
| **3** | - Chạy kịch bản thử nghiệm dồn tải giả lập 50 file tải lên đồng thời.<br>- Phân tích hiện tượng Lambda scale ồ ạt làm ảnh hưởng hạn mức concurrency của tài khoản. | 01/09/2026 | 01/09/2026 | [Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html) |
| **4** | - Đưa Amazon SQS vào giữa làm bộ đệm điều tiết dòng chảy dữ liệu.<br>- Cấu hình MaximumConcurrency = 5 trên Event Source Mapping để khóa trần scaling. | 02/09/2026 | 02/09/2026 | [SQS as Event Source](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html) |
| **5** | - Kích hoạt cơ chế ReportBatchItemFailures để chỉ retry các tin nhắn bị lỗi trong batch.<br>- Thiết lập SQS Dead-Letter Queue (DLQ) với maxReceiveCount = 3. | 03/09/2026 | 03/09/2026 | [Batch Item Failures](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html#services-sqs-batchfailurereporting) |
| **6** | - Phát hiện lỗi tin nhắn bị xử lý lặp do thời gian chờ SQS ngắn hơn thời gian chạy của Lambda.<br>- Điều chỉnh SQS Visibility Timeout lên 360 giây (gấp 6 lần Lambda timeout 60s). | 04/09/2026 | 04/09/2026 | [SQS Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) |

### 3. Kết quả đạt được tuần 3:
* Chuyển đổi thành công pipeline tiếp nhận từ mô hình Đẩy (Push) sang mô hình Kéo (Pull) qua SQS, xử lý an toàn đợt tải 50 file đồng thời mà không làm tăng vọt concurrency.
* Khóa trần concurrency của hàm Ingestion ở mức tối đa 5 workers song song thông qua MaximumConcurrency, bảo vệ downstream services khỏi nguy cơ quá tải.
* Ứng dụng ReportBatchItemFailures giúp xóa ngay các item thành công và chỉ retry đúng item bị lỗi, tiết kiệm tài nguyên xử lý.
* Xử lý triệt để lỗi duplicate processing: phân tích nguyên nhân do Visibility Timeout mặc định 30s ngắn hơn Lambda timeout 60s khiến tin nhắn bị requeue sớm; nâng Visibility Timeout lên 360s giải quyết dứt điểm vấn đề.
* Lưới an toàn Dead-Letter Queue (DLQ) được thiết lập với thời gian lưu trữ tin nhắn lỗi tối đa 14 ngày.

### 4. Tài liệu tham khảo:
* [Using AWS Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
* [Amazon SQS Visibility Timeout Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
* [AWS Lambda Concurrency Management](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
