---
title: "Tuần 4"
date: 2026-09-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Tuần 4: Khám Phá Amazon Rekognition & Transcribe
*Thời gian thực hiện: 07/09/2026 – 11/09/2026*

### 1. Mục tiêu tuần 4:
* Khảo sát các API Computer Vision của Amazon Rekognition (DetectLabels, DetectText) trên tập ảnh slide bài giảng và ảnh chụp bảng viết tay.
* Đánh giá độ chính xác bóc tách giọng nói tiếng Việt (vi-VN) bằng Amazon Transcribe qua API StartTranscriptionJob.
* Đo lường thời gian xử lý và kích thước payload của các định dạng media để chuẩn bị dữ liệu thực tế cho bảng tính chi phí FinOps.
* Khắc phục giới hạn kích thước tệp khi truyền dữ liệu trực tiếp vào Amazon Rekognition.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Viết script Python Boto3 gọi thử nghiệm API DetectLabels của Amazon Rekognition.<br>- Đánh giá độ chính xác phân loại đối tượng trên 20 ảnh slide bài giảng CNTT. | 07/09/2026 | 07/09/2026 | [Rekognition Labels](https://docs.aws.amazon.com/rekognition/latest/dg/labels.html) |
| **3** | - Thử nghiệm API DetectText bóc tách ký tự tiếng Việt và chữ viết bảng.<br>- Khảo sát ngưỡng độ tin cậy để loại bỏ các từ bị nhận diện sai lệch. | 08/09/2026 | 08/09/2026 | [Rekognition Text](https://docs.aws.amazon.com/rekognition/latest/dg/text-detection.html) |
| **4** | - Gọi thử nghiệm Amazon Transcribe qua API StartTranscriptionJob với mã ngôn ngữ vi-VN.<br>- Bóc băng 5 đoạn ghi âm bài giảng (3–5 phút) và đo đếm thời gian chuyển đổi văn bản. | 09/09/2026 | 09/09/2026 | [Amazon Transcribe](https://docs.aws.amazon.com/transcribe/latest/dg/) |
| **5** | - Phân tích giới hạn kích thước tệp: phát hiện Rekognition từ chối ảnh > 5MB nếu truyền qua tham số Bytes.<br>- Chuyển sang truyền tham số S3Object để hỗ trợ ảnh dung lượng lên đến 15MB. | 10/09/2026 | 10/09/2026 | [Rekognition Limits](https://docs.aws.amazon.com/rekognition/latest/dg/limits.html) |
| **6** | - Tổng hợp bảng số liệu thực nghiệm về thời gian xử lý và kích thước payload của các định dạng media.<br>- Chuẩn bị dữ liệu thực tế cho bảng tính chi phí FinOps. | 11/09/2026 | 11/09/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

### 3. Kết quả đạt được tuần 4:
* Nắm vững các API cốt lõi của Rekognition và Transcribe vi-VN, đánh giá khách quan độ chính xác nhận diện chữ viết bảng và giọng nói chuyên ngành kỹ thuật.
* Khắc phục triệt để lỗi giới hạn kích thước tệp: phát hiện Rekognition giới hạn 5MB khi truyền payload dạng Bytes base64; chuyển sang truyền đối tượng tham chiếu S3 (S3Object) giúp hỗ trợ ảnh lên đến 15MB an toàn.
* Thiết lập mốc độ tin cậy tối ưu: Confidence >= 65.0 cho nhãn hình ảnh và >= 70.0 cho văn bản OCR, loại bỏ hoàn toàn các ký tự rác.
* Xây dựng xong bảng số liệu thực nghiệm đo đạc độ trễ và chi phí làm cơ sở thiết kế hệ thống.

### 4. Tài liệu tham khảo:
* [Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html)
* [Amazon Transcribe Developer Guide](https://docs.aws.amazon.com/transcribe/latest/dg/what-is-transcribe.html)
* [Amazon Rekognition Service Quotas](https://docs.aws.amazon.com/general/latest/gr/rekognition.html)
