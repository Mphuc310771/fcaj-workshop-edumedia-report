---
title: "Tự đánh giá"
date: 2026-11-05
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# 6. Tự Đánh Giá Kết Quả Thực Tập (Self-Evaluation)

**Sinh viên thực hiện:** Đỗ Trần Minh Phúc  
**Trường:** Đại học Khoa học Tự nhiên, ĐHQG-HCM (HCMUS)  
**Chương trình:** First Cloud AI Journey (FCAJ) — AWS Vietnam  
**Dự án:** EduMedia AI - Smart Lecture Storage & Search Engine (Hệ thống Lưu trữ & Tra cứu Bài giảng Thông minh trên AWS)

---

## 1. Bảng Đánh Giá Chi Tiết Theo 8 Khung Tiêu Chí Quy Định

| Tiêu chí đánh giá | Xếp loại | Nhận xét cụ thể & Minh chứng thực tế |
| :--- | :---: | :--- |
| **1. Kiến thức chuyên môn** | **Tốt** | Nắm vững và làm chủ hệ sinh thái AWS Serverless & GenAI: S3, SQS Buffer, Lambda, Rekognition, Transcribe, DynamoDB, Bedrock, CloudWatch. |
| **2. Khả năng tự học & nghiên cứu** | **Tốt** | Nhanh chóng tiếp thu công nghệ mới từ tài liệu chính thức của AWS; làm chủ SAM Template, xử lý vector embedding và nguyên lý FinOps Cost Optimization. |
| **3. Tính chủ động trong công việc** | **Tốt** | Tự xây dựng bộ kiểm thử unit test tự động với Pytest (143 tests), thiết lập phân quyền IAM Least-Privilege, xây dựng Runbook dọn dẹp tài nguyên và cấu hình Dead-Letter Queue (DLQ). |
| **4. Kỷ luật & tuân thủ quy định** | **Tốt** | Chấp hành đúng nội quy chương trình, tham gia đầy đủ các buổi làm việc tại văn phòng AWS Tầng 26 Bitexco theo lịch quy định và nộp báo cáo đúng hạn (<mark style="background-color: #fef08a; color: #854d0e; padding: 2px 6px; border-radius: 4px; font-weight: 600; border: 1px solid #fde047;">01 buổi nghỉ ốm ngày 27/08/2026 có đính kèm toa thuốc tại <a href="../1-worklog/1.2-week2/#4-minh-chứng-nghỉ-ốm-ngày-27082026" style="text-decoration: underline; font-weight: bold; color: #854d0e;">Tuần 2</a></mark>). |
| **5. Kỹ năng giao tiếp** | **Tốt** | Soạn thảo tài liệu và báo cáo song ngữ (Anh - Việt) đầy đủ, trình bày giải pháp kiến trúc rõ ràng, chủ động trao đổi chuyên môn và chia sẻ kinh nghiệm kỹ thuật với các bạn sinh viên trong chương trình. |
| **6. Làm việc nhóm (Teamwork)** | **Tốt** | Phối hợp hiệu quả với bạn thực tập cùng nhóm trong việc rà soát chéo các bài blog kỹ thuật, trao đổi giải pháp thiết kế kiến trúc và chia sẻ kinh nghiệm học tập. |
| **7. Năng lực giải quyết vấn đề** | **Tốt** | Xử lý triệt để lỗi đệ quy S3-Transcribe bằng tách prefix `uploads/` và `transcripts/`; tối ưu vector 1536 ➔ 256 chiều; tự động S3-offloading transcript > 250KB chống vượt ngưỡng 400KB DynamoDB. |
| **8. Đóng góp cho Workshop** | **Tốt** | Xây dựng trọn vẹn tài liệu Workshop 7 phần có tính ứng dụng cao; bổ sung hệ thống Guardrails bảo vệ dữ liệu (chặn xóa HTTP 403, giới hạn upload ≤ 25MB) và giao diện Web UI tương tác với Click-to-Seek Media Player. |

---

## 2. Những Khó Khăn Đã Vượt Qua

1. **Xử lý tác vụ ASR bất đồng bộ của Transcribe:** Ban đầu, việc đợi Transcribe hoàn thành trong hàm Lambda dễ gây timeout (120s). Tôi đã khắc phục bằng cách thiết kế kiến trúc phân tách trạng thái INDEXING → READY kết hợp SQS Buffer và EventBridge.
2. **Tối ưu hóa chi phí Vector Embeddings:** Việc lưu trữ vector 1536 chiều trên DynamoDB làm tăng dung lượng bảng. Tôi đã áp dụng kỹ thuật **Titan Vector Subsampling** rút gọn về 256 chiều, giảm 83% chi phí lưu trữ mà vẫn giữ độ chính xác trên 98%.
3. **Kiểm soát An toàn Dữ liệu & Giới hạn 400KB DynamoDB:** Tôi đã chủ động thiết kế cơ chế Guardrails bảo vệ dữ liệu (chặn xóa file gốc với mã 403, giới hạn dung lượng upload ≤ 25MB) và cơ chế tự động chuyển transcript > 250KB sang S3 để triệt tiêu hoàn toàn rủi ro ValidationException của DynamoDB.

---

## 3. Định Hướng Phát Triển Tiếp Theo

* Xây dựng thêm CI/CD Pipeline tự động triển khai bằng **AWS CodePipeline / GitHub Actions**.
* Đóng gói và phân phối Web Frontend qua **Amazon CloudFront + S3 Static Hosting** kết hợp xác thực người dùng bằng **Amazon Cognito**.
