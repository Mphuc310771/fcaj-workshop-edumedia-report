---
title: "Tuần 11"
date: 2026-10-30
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

# Tuần 11: Kiểm Thử Tự Động 143 Tests, Giám Sát CloudWatch & Viết Blog
*Thời gian thực hiện: 26/10/2026 – 30/10/2026*

### 1. Mục tiêu tuần 11:
* Xây dựng bộ kiểm thử tự động toàn diện bằng Pytest, mock các dịch vụ AWS bằng moto và unittest.mock, đạt 143 tests passing với độ bao phủ mã nguồn 83.69%.
* Chạy bài thử nghiệm tải dồn dập (Stress Benchmark) 50 files đồng thời để kiểm chứng khả năng phục hồi và cơ chế đệm SQS của hệ thống.
* Thiết lập 7 CloudWatch Alarms và SNS Topic agentsys-alerts-topic phục vụ giám sát vận hành chuẩn Production.
* Soạn thảo 2 bài viết Blog kỹ thuật cá nhân đúc kết từ quá trình kiến trúc và tối ưu hệ thống (S3-SQS-Lambda, DynamoDB GSI) và chia sẻ lên cộng đồng AWS Study Group.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Thiết lập cấu trúc thư mục kiểm thử tests/.<br>- Viết unit tests cho SQS consumer và router bằng moto mock S3/DynamoDB. | 26/10/2026 | 26/10/2026 | [Moto Library](http://docs.getmoto.org/) |
| **3** | - Mở rộng test suite kiểm tra các edge cases: file corrupt, Bedrock timeout, offloading > 250KB.<br>- Đạt mốc 143 tests pass với 83.69% code coverage. | 27/10/2026 | 27/10/2026 | [Pytest-cov Guide](https://pytest-cov.readthedocs.io/) |
| **4** | - Cấu hình 7 CloudWatch Alarms trên SAM template, kích hoạt SNS alert topic.<br>- Phát các chỉ số tùy biến (Custom Metrics) MediaIngested/MediaIngestFailed và chạy stress test 50 uploads đồng thời. | 28/10/2026 | 28/10/2026 | [CloudWatch Alarms Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) |
| **5** | - Soạn thảo bài viết Blog 1: Phân tích tại sao không để S3 trigger trực tiếp Lambda, cơ chế Claim Check và DLQ.<br>- Hoàn thiện bài Blog 2: Phân tích cơ chế vật lý của DynamoDB Scan và giải pháp GSI. | 29/10/2026 | 29/10/2026 | blog_1/bai_viet.txt, blog_2/bai_viet.txt |
| **6** | - Hoàn thiện định dạng các bài viết và đăng tải lên cộng đồng AWS Study Group.<br>- Tương tác thảo luận và giải đáp phản hồi từ các thành viên cộng đồng. | 30/10/2026 | 30/10/2026 | [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) |

### 3. Kết quả đạt được tuần 11:
* Xây dựng bộ kiểm thử hoàn chỉnh với 143 unit test cases đạt 83.69% code coverage, đảm bảo không phát sinh lỗi hồi quy trên toàn hệ thống.
* Thiết lập 7 CloudWatch Alarms và SNS Topic sẵn sàng kích hoạt cảnh báo thời gian thực khi có lỗi runtime hoặc có tin nhắn rơi vào DLQ.
* Chạy kịch bản stress test tải 50 files đồng thời thành công, xác nhận tính đàn hồi và khả năng chịu tải của kiến trúc SQS sang Lambda.
* Hoàn thành và đăng tải 2 bài blog chuyên sâu cá nhân (Blog 1 & 2) lên cộng đồng AWS Study Group.

### 4. Tài liệu tham khảo:
* [Testing AWS Lambda with Moto](http://docs.getmoto.org/en/latest/)
* [Amazon CloudWatch Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
* [AWS Study Group Facebook Community](https://www.facebook.com/groups/awsstudygroupfcj)
* [Amazon DynamoDB Core Components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
