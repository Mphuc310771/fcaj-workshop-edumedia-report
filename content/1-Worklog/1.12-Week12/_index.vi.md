---
title: "Tuần 12"
date: 2026-11-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

# Tuần 12: Đóng Gói Tài Liệu Workshop 7 Phần & Nghiệm Thu Kỹ Thuật
*Thời gian thực hiện: 02/11/2026 – 06/11/2026*

### 1. Mục tiêu tuần 12:
* Biên soạn và đóng gói trọn vẹn bộ tài liệu hướng dẫn thực hành Workshop 7 phần (từ Chương 5.1 đến 5.7) theo tiêu chuẩn FCAJ.
* Xây dựng và kiểm thử hoàn chỉnh kịch bản dọn dẹp tài nguyên (Clean-up runbook và script `cleanup.sh`) đảm bảo xóa sạch môi trường phòng ngừa phát sinh chi phí ngoài ý muốn.
* Đối chiếu bảng dự toán FinOps trên AWS Pricing Calculator và số liệu thực tế trên AWS Cost Explorer / Budgets ($0.00 USD).
* Rà soát toàn bộ mã nguồn trên GitHub, hoàn thiện báo cáo thực tập tổng kết và tiến hành nghiệm thu kỹ thuật dự án EduMedia AI.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Biên soạn nội dung Workshop Chương 5.1 (Tổng quan kiến trúc & FinOps) và Chương 5.2 (Chuẩn bị môi trường & AWS SAM CLI).<br>- Trình bày các bước cấu hình tài khoản AWS và công cụ phát triển. | 02/11/2026 | 02/11/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) |
| **3** | - Biên soạn Workshop Chương 5.3 (Triển khai Hạ tầng IaC SAM) và Chương 5.4 (Kiểm thử chức năng trích xuất Media Rekognition & Transcribe).<br>- Chụp ảnh minh họa từng bước triển khai CloudFormation và luồng trích xuất dữ liệu. | 03/11/2026 | 03/11/2026 | [Amazon Rekognition Docs](https://docs.aws.amazon.com/rekognition/) |
| **4** | - Biên soạn Workshop Chương 5.5 (Tra cứu thông minh Bedrock & Web Gallery) và Chương 5.7 (Phân tích đánh đổi kỹ thuật & Định hướng mở rộng).<br>- Hướng dẫn thao tác trải nghiệm tìm kiếm ngữ nghĩa và tra cứu tài liệu học tập. | 04/11/2026 | 04/11/2026 | [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/) |
| **5** | - Xây dựng nội dung Chương 5.6 (Giám sát CloudWatch & Kịch bản dọn dẹp Cleanup).<br>- Viết và kiểm thử script tự động `cleanup.sh` xóa sạch S3 bucket, DynamoDB table và SAM stack trong 3 phút. | 05/11/2026 | 05/11/2026 | [AWS CLI S3 cleanup](https://docs.aws.amazon.com/cli/latest/reference/s3/rm.html) |
| **6** | - Đối chiếu chi phí thực tế trên AWS Budgets / Cost Explorer; rà soát toàn bộ tài liệu báo cáo.<br>- Kiểm tra lại kho mã nguồn trên GitHub và hoàn tất nghiệm thu kỹ thuật dự án thực tập EduMedia AI. | 06/11/2026 | 06/11/2026 | [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) |

### 3. Kết quả đạt được tuần 12:
* Hoàn thiện 100% tài liệu Workshop 7 phần từ 5.1 đến 5.7 với hướng dẫn từng bước chi tiết, mạch lạc và kèm ảnh minh họa thực tế.
* Script `cleanup.sh` và Runbook dọn dẹp tài nguyên được kiểm thử thành công, đảm bảo xóa sạch môi trường chỉ trong 3 phút, phòng tránh phát sinh chi phí ngoài ý muốn.
* Chi phí thực tế được đối chiếu minh bạch đạt $0.00 USD, khớp chính xác với nguyên lý Scale-to-Zero.
* Toàn bộ mã nguồn và tài liệu hướng dẫn được đóng gói hoàn chỉnh trên GitHub, hoàn tất trọn vẹn kỳ thực tập FCAJ.

### 4. Tài liệu tham khảo:
* [AWS CloudFormation Stack Deletion](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html)
* [AWS Well-Architected Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/)
* [AWS Pricing Calculator - AgentSys Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
