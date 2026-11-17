---
title: "Tuần 1"
date: 2026-08-21
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Tuần 1: Khởi Động Chương Trình FCAJ & Thiết Lập Môi Trường
*Thời gian thực hiện: 17/08/2026 – 21/08/2026*

### 1. Mục tiêu tuần 1:
* Tham gia tiếp nhận chương trình thực tập First Cloud AI Journey (FCAJ 2026), đăng ký tài khoản HCM-Portal và nắm các quy định văn phòng.
* Cài đặt bộ công cụ phát triển: AWS CLI v2, AWS SAM CLI, Python 3.12 và Docker Engine trên máy cục bộ Ubuntu 24.04.
* Thiết lập tài khoản AWS, kích hoạt bảo mật MFA và cấu hình AWS CLI named profile với quyền IAM phù hợp.
* Khởi tạo cấu trúc repository dự án EduMedia AI trên GitHub và phác thảo hướng tiếp cận Serverless cho bài toán xử lý media.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Tham gia buổi onboarding chương trình thực tập FCAJ 2026.<br>- Kích hoạt tài khoản HCM-Portal, tìm hiểu lộ trình đào tạo và các quy định về chuyên cần, tác phong. | 17/08/2026 | 17/08/2026 | [Nội quy & Hướng dẫn FCAJ HCM](https://hcm-rules.awsfcaj.com/) |
| **3** | - Cài đặt môi trường phát triển trên Ubuntu 24.04: Python 3.12, AWS CLI v2, AWS SAM CLI và Docker.<br>- Kiểm tra hoạt động của Docker daemon để sẵn sàng test Lambda cục bộ. | 18/08/2026 | 18/08/2026 | [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/) |
| **4** | - Thiết lập tài khoản AWS, tạo IAM User quản trị riêng biệt (không dùng Root Account), bật bảo mật MFA.<br>- Cấu hình named profile trong ~/.aws/credentials và kiểm tra kết nối qua lệnh aws sts get-caller-identity. | 19/08/2026 | 19/08/2026 | [AWS Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) |
| **5** | - Khởi tạo repository dự án EduMedia AI trên GitHub, thiết lập khung thư mục mã nguồn (infra, src, tests).<br>- Soạn thảo cấu trúc tài liệu kỹ thuật song ngữ cho dự án. | 20/08/2026 | 20/08/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/) |
| **6** | - Khảo sát các dịch vụ AWS phục vụ bài toán đa phương tiện: S3 (lưu trữ), Lambda (xử lý), Rekognition & Transcribe (AI bóc tách), DynamoDB & Bedrock (tra cứu).<br>- Phác thảo sơ bộ luồng xử lý Event-Driven bất đồng bộ cho EduMedia AI. | 21/08/2026 | 21/08/2026 | [AWS Serverless Architecture](https://aws.amazon.com/serverless/) |

### 3. Kết quả đạt được tuần 1:
* Kích hoạt thành công tài khoản HCM-Portal và nắm vững lịch trình làm việc tại văn phòng cũng như các mốc đánh giá của kỳ thực tập.
* Hoàn tất thiết lập môi trường phát triển cục bộ, cấu hình AWS CLI credentials trỏ về region us-east-1.
* Bảo mật tài khoản AWS bằng MFA và sử dụng IAM User riêng biệt thay vì Root Account để thao tác hàng ngày theo khuyến nghị của AWS.
* Khởi tạo xong repository dự án trên GitHub với đầy đủ cấu trúc thư mục mã nguồn và tài liệu kỹ thuật ban đầu.
* Định hình kiến trúc cốt lõi cho EduMedia AI: chọn mô hình Serverless Pay-as-you-go (Lambda, S3, DynamoDB On-Demand) giúp hệ thống tự động scale-to-zero khi không có yêu cầu, tối ưu chi phí vận hành và không tốn tiền duy trì server nhàn rỗi.

### 4. Tài liệu tham khảo:
* [Cổng Nội quy & Hướng dẫn FCAJ HCM](https://hcm-rules.awsfcaj.com/)
* [AWS Command Line Interface User Guide v2](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
* [AWS Serverless Application Model (SAM) Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
* [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
* [AWS Serverless Architecture Overview](https://aws.amazon.com/serverless/)
