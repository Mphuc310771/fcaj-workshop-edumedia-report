---
title: "Tuần 5"
date: 2026-09-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Tuần 5: Thiết Kế Kiến Trúc AgentSys & Hoàn Thiện Proposal
*Thời gian thực hiện: 14/09/2026 – 18/09/2026*

### 1. Mục tiêu tuần 5:
* Hoàn thiện sơ đồ kiến trúc tổng thể của AgentSys trên Draw.io theo chuẩn AWS Architecture Icons 2026.
* Soạn thảo bản Đề xuất Dự án (Proposal) 7 phần đầy đủ theo hướng dẫn của ban tổ chức FCAJ.
* Lập mô hình dự toán chi phí chi tiết bằng công cụ chính thức AWS Pricing Calculator (calculator.aws).
* Đánh giá kiến trúc đối chiếu với 6 trụ cột của AWS Well-Architected Framework.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Thiết kế sơ đồ kiến trúc hệ thống AgentSys trên Draw.io.<br>- Sử dụng bộ icon AWS Architecture 2026 mới nhất, chuẩn hóa các luồng dữ liệu 5 tầng. | 14/09/2026 | 14/09/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| **3** | - Soạn thảo các phần bối cảnh, bài toán thực tế (Problem Statement) và phạm vi triển khai.<br>- Định nghĩa rõ các tiêu chuẩn hoàn thành cho từng dịch vụ AWS. | 15/09/2026 | 15/09/2026 | Mẫu Proposal FCAJ |
| **4** | - Lập bảng dự toán chi phí chi tiết trên AWS Pricing Calculator tại region us-east-1.<br>- Chứng minh hệ thống đạt chi phí 0 USD khi ở trạng thái nghỉ (Scale-to-Zero). | 16/09/2026 | 16/09/2026 | [AWS Pricing Calculator - AgentSys Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934) |
| **5** | - Viết phần phân tích đối chiếu 6 trụ cột AWS Well-Architected Framework: Security, Reliability, Performance, Cost, Operations, Sustainability.<br>- Bổ sung cơ chế bảo mật IAM Least Privilege và S3 SSE-S3. | 17/09/2026 | 17/09/2026 | [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/) |
| **6** | - Hoàn thiện tài liệu Proposal song ngữ và lưu trữ vào repository báo cáo tại content/2-Proposal/.<br>- Rà soát các tiêu chí kỹ thuật chuẩn bị cho giai đoạn xây dựng mã nguồn. | 18/09/2026 | 18/09/2026 | Quy định Proposal FCAJ |

### 3. Kết quả đạt được tuần 5:
* Hoàn thiện sơ đồ kiến trúc AgentSys chi tiết trên Draw.io thể hiện rõ 5 tầng: Upload, Ingestion, AI Extraction, Storage & Querying.
* Soạn thảo bản Đề xuất Dự án (Proposal) song ngữ hoàn chỉnh tại content/2-Proposal/ với đầy đủ 8 mục nội dung.
* Xây dựng bảng dự toán chi phí trên AWS Pricing Calculator: chi phí xử lý 1 ảnh khoảng 0.00207 USD (~52 VNĐ) và 1 phút audio khoảng 0.02435 USD (~610 VNĐ); mô hình Pay-as-you-go giúp tối ưu ngân sách theo nhu cầu thực tế và tự co về 0 khi không có tải.
* Phân tích thấu đáo 6 trụ cột Well-Architected: bảo vệ dữ liệu với S3 Block Public Access, chống nghẽn với SQS load leveling, tối ưu vector 256 chiều và Scale-to-Zero khi không có truy cập.
* Sẵn sàng các tài liệu thiết kế kiến trúc để bước vào giai đoạn hiện thực hóa mã nguồn ở tuần tiếp theo.

### 4. Tài liệu tham khảo:
* [AWS Architecture Center & Official Icons](https://aws.amazon.com/architecture/icons/)
* [AWS Well-Architected Framework Whitepapers](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
* [AWS Pricing Calculator - AgentSys Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
