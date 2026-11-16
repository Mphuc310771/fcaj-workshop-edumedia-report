---
title: "Workshop"
date: 2026-10-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# EduMedia AI: Hệ Thống Lưu Trữ & Tra Cứu Bài Giảng Thông Minh Trên AWS

Workshop này hướng dẫn bạn từng bước xây dựng và triển khai toàn bộ hệ thống **EduMedia AI** bằng **AWS Serverless Application Model (AWS SAM)**.

Sau khi hoàn thành workshop, bạn sẽ sở hữu một hệ thống hoàn chỉnh:
* Người dùng tải ảnh, âm thanh, video lên **Amazon S3** an toàn qua **Presigned URL**.
* Sự kiện tải lên được đệm qua **Amazon SQS & DLQ**, kích hoạt **AWS Lambda**.
* Tự động nhận diện nhãn đồ vật và OCR văn bản qua **Amazon Rekognition**; bóc băng giọng nói tiếng Việt (*vi-VN*) qua **Amazon Transcribe**.
* Trích xuất và nhúng vector ngữ nghĩa qua **Amazon Bedrock (Amazon Titan Text Embeddings V2)** và lưu trữ trên **Amazon DynamoDB**.
* Tìm kiếm ngữ nghĩa thông minh bằng câu hỏi tiếng Việt tự nhiên qua **Amazon Bedrock (Nova Lite)**.
* Giám sát vận hành tự động qua **Amazon CloudWatch Logs & Alarms**.

---

### Danh Sách Các Bài Lab Thực Hành:

1. **[5.1. Tổng quan bài toán, Kiến trúc Serverless & Ước tính FinOps](5.1-Workshop-overview/)**
2. **[5.2. Chuẩn bị môi trường, AWS CLI, SAM, IAM Least-Privilege & Bedrock Model Access](5.2-Prerequisite/)**
3. **[5.3. Thiết lập Amazon S3, SQS Buffer Queue, Dead-Letter Queue (DLQ) & Event Notifications](5.3-S3-Storage/)**
4. **[5.4. Triển khai Lambda Ingestion Engine với Amazon Rekognition & Transcribe vi-VN](5.4-AI-Pipeline/)**
5. **[5.5. Xây dựng Trợ lý Tìm kiếm với Amazon Bedrock & DynamoDB GSI Catalog](5.5-Bedrock-Search/)**
6. **[5.6. Thiết lập CloudWatch Monitoring, Alarms và Quy trình Dọn dẹp Tài nguyên (Cleanup)](5.6-Monitoring-Cleanup/)**
7. **[5.7. Thảo luận: Kiến trúc Đánh đổi & Lộ trình Nâng cấp Enterprise](5.7-Architecture-Tradeoffs/)**

---

* **Thời lượng ước tính:** 90 – 120 phút.
* **Khu vực triển khai (AWS Region):** us-east-1 (N. Virginia).
* **Ước tính chi phí thực hành:** **<$0.05 USD** (Kiến trúc Serverless tính theo lượt dùng; với số lượng tệp lab mẫu, chi phí phát sinh hầu như không đáng kể).
* **File đính kèm:** [SAM Template chính thức (template.yaml)](/files/agentsys-sam-template.yaml)
