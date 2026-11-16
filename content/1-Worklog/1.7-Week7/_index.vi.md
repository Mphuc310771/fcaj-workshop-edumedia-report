---
title: "Tuần 7"
date: 2026-10-02
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Tuần 7: Tích Hợp Rekognition & Xử Lý Async Transcribe
*Thời gian thực hiện: 28/09/2026 – 02/10/2026*

### 1. Mục tiêu tuần 7:
* Tích hợp Amazon Rekognition (DetectLabels, DetectText) vào hàm Lambda Ingest để tự động bóc tách nhãn và văn bản tiếng Việt từ ảnh bài giảng.
* Thiết kế mô hình xử lý bất đồng bộ (Asynchronous Orchestration) cho Amazon Transcribe: worker đẩy job và gán trạng thái INDEXING.
* Cấu hình Amazon EventBridge Rule bắt sự kiện Transcribe Job State Change (COMPLETED) để kích hoạt Lambda hoàn tất bóc băng.
* Xây dựng thuật toán băm nhỏ bài giảng âm thanh thành các phân đoạn (chunks) có gắn mốc thời gian (start_time, end_time).

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Tích hợp logic gọi Rekognition OCR và trích xuất nhãn trong apps/workers/ingestion.py.<br>- Áp dụng ngưỡng độ tin cậy tối thiểu Confidence >= 65.0 để lọc bỏ các từ nhiễu. | 28/09/2026 | 28/09/2026 | [Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/) |
| **3** | - Xây dựng cơ chế phân tách trạng thái cho file âm thanh/video.<br>- Worker khởi tạo start_transcription_job và lưu bản ghi DynamoDB ở trạng thái INDEXING rồi thoát ngay. | 29/09/2026 | 29/09/2026 | [Transcribe Job Status](https://docs.aws.amazon.com/transcribe/latest/dg/how-it-works.html) |
| **4** | - Cấu hình EventBridge Rule trên SAM template.<br>- Lắng nghe sự kiện Transcribe Job State Change với trạng thái COMPLETED và định tuyến về Lambda xử lý. | 30/09/2026 | 30/09/2026 | [EventBridge with Transcribe](https://docs.aws.amazon.com/transcribe/latest/dg/monitoring-events.html) |
| **5** | - Hiện thực thuật toán băm nhỏ transcript thành các phân đoạn 30–60 giây.<br>- Gắn kèm mốc start_time và end_time cho từng phân đoạn để hỗ trợ tính năng đồng bộ media player. | 01/10/2026 | 01/10/2026 | Python Text Chunking Patterns |
| **6** | - Chạy kiểm thử end-to-end với file bài giảng âm thanh 10 phút.<br>- Kiểm tra việc tự động chuyển đổi trạng thái từ INDEXING sang READY khi EventBridge nhận tín hiệu. | 02/10/2026 | 02/10/2026 | [AWS EventBridge Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) |

### 3. Kết quả đạt được tuần 7:
* Hoàn thiện pipeline bóc tách nhãn và chữ viết tiếng Việt tự động cho toàn bộ hình ảnh tải lên, tổng hợp thành trường mô tả phục vụ tìm kiếm.
* Thiết lập thành công cơ chế Asynchronous EventBridge cho Transcribe: worker ban đầu chỉ mất ~1.2 giây compute để khởi tạo job rồi thoát ngay, tiết kiệm 95% chi phí thời gian chờ compute của Lambda.
* Thuật toán băm transcript kèm mốc thời gian hoạt động ổn định trên các bài giảng dài 10–30 phút, tạo tiền đề cho tính năng nghe đúng đoạn giảng viên đang nói.
* Trạng thái tệp chuyển đổi mượt mà từ INDEXING sang READY khi EventBridge nhận tín hiệu COMPLETED.

### 4. Tài liệu tham khảo:
* [Amazon EventBridge Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)
* [Monitoring Transcribe with Amazon EventBridge](https://docs.aws.amazon.com/transcribe/latest/dg/monitoring-events.html)
* [Amazon Rekognition Text Detection Guide](https://docs.aws.amazon.com/rekognition/latest/dg/text-detection.html)
