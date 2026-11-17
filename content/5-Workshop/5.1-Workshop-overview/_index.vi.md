---
title: "Tổng quan"
date: 2026-10-02
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# 5.1. Tổng Quan Bài Toán, Kiến Trúc Serverless & Ước Tính FinOps

---

## 1. Bài toán thực tế (Problem Statement)

Người dùng (học sinh, sinh viên, kỹ sư) lưu trữ hàng nghìn hình ảnh bài giảng, ảnh chụp bảng viết tay, tệp ghi âm giọng nói dài và video bài học trong thiết bị. 
* Tên file mặc định (IMG\_091522.jpg, voice\_record\_01.m4a) không chứa bất kỳ ngữ nghĩa nào.
* Việc tìm lại nội dung như *"Đoạn thầy nhắc làm bài tập ma trận"* hay *"Công thức tính tích phân trên bảng"* thường mất hàng giờ xem lại thủ công.

---

## 2. Mục tiêu Workshop (Workshop Objectives)

Xây dựng hệ thống **EduMedia AI** xử lý tự động theo mô hình **Event-Driven Serverless**:
1. Upload tệp trực tiếp lên **Amazon S3** (Private Bucket, mã hóa SSE-S3).
2. Tự động đệm sự kiện qua **Amazon SQS** và **Dead-Letter Queue (DLQ)**.
3. **AWS Lambda** điều phối phân tích:
   * Ảnh: **Amazon Rekognition** trích xuất nhãn đối tượng + OCR văn bản tiếng Việt.
   * Âm thanh / Video: **Amazon Transcribe** bóc băng giọng nói tiếng Việt (*vi-VN*).
4. Nhúng vector 256 chiều qua **Amazon Titan Text Embeddings V2** và lưu catalog vào **Amazon DynamoDB**.
5. Nhận diện và trả lời câu hỏi tự nhiên tiếng Việt qua **Amazon Bedrock (Nova Lite)**.
6. Giám sát toàn diện qua **Amazon CloudWatch Logs & Alarms**.

---

## 3. Sơ đồ Kiến trúc Hệ thống (System Architecture)

![Kiến trúc Hệ thống EduMedia AI trên AWS](/images/5-Workshop/architecture.png)


---

## 4. Dự toán Chi phí Vận hành FinOps (Mô hình Workload Sản xuất & Chi phí Thực tế)

Trong khuôn khổ dự án FCAJ, chi phí vận hành được tính toán theo nguyên lý **FinOps (Cloud Financial Management)** dựa trên 2 kịch bản thực tế đối chiếu qua **AWS Pricing Calculator**:
* **Link dự toán chính thức trên AWS:** [Bảng dự toán công khai EduMedia AI](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)

### 4.1. Kịch bản 1: Người đọc tự triển khai thực hành Workshop (1 Tester / Self-hosted Lab)
* **Khối lượng kiểm thử:** 50 ảnh slide học tập, 5 tệp âm thanh (20 phút bài giảng), 100 lượt truy vấn Bedrock AI, 1 GB lưu trữ S3, 2.000 invocations Lambda & DynamoDB.
* **Chi phí gốc tiêu thụ (Gross Cost - khi không có Free Tier):** **$0.48 – $0.84 USD** (~12.000 – 21.000 VNĐ cho toàn bộ bài lab).
* **Hóa đơn thực tế (Net Cost - khi có Free Tier):** Chỉ trả phần Bedrock On-Demand khoảng **$0.18 USD (~4.500 VNĐ)**, hoặc **$0.00 USD** nếu dùng tài khoản mới có gói tín dụng AWS Free Tier Credits.

### 4.2. Kịch bản 2: Triển khai thử nghiệm cho một lớp học (100 – 200 Sinh viên / 1 Tháng)
Khối lượng phục vụ: 1.500 ảnh bài giảng/slide, 250 phút ghi âm bài giảng dùng chung, 2.000 lượt tra cứu Bedrock, 20 GB lưu trữ S3, 15.000 invocations Lambda, DynamoDB On-Demand, 10 GB CloudFront DTO.

| Dịch vụ AWS | Khối lượng vận hành (100 - 200 sinh viên) | Gross Cost (Không Free Tier) | Hóa đơn Chuẩn dài hạn (Áp dụng Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1.500 ảnh × 2 APIs = 3.000 calls | **$3.00 USD** | **$3.00 USD** *(Năm đầu: $2.00)* |
| **Amazon Transcribe** | 250 phút ghi âm bài giảng vi-VN | **$1.50 USD** *(giá cũ: $6.00)* | **$1.50 USD** *(giá cũ: $6.00)* |
| **Amazon Bedrock** | 2.000 queries Nova Pro (1K in + 300 out) + Embed | **$3.56 USD** *(Haiku: $4.04)* | **$3.56 USD** *(Haiku: $4.04)* |
| **Amazon S3** | 20 GB Standard tích lũy + requests | **$0.46 USD** *(+ requests: $0.53)* | **$0.46 USD** *(Năm đầu: $0.35)* |
| **AWS Lambda & SQS** | 15.000 invocations (22.500 GB-s) + SQS | **$0.38 USD** | **$0.00 USD** *(Always Free Tier)* |
| **Amazon DynamoDB** | On-Demand (15.000 writes + 60.000 reads) | **$0.02 USD** | **$0.02 USD** *(Lưu trữ < 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB/tháng)* |
| **TỔNG CỘNG** | **Toàn bộ hệ thống cho 100 - 200 Sinh viên** | **$9.77 – $14.27 USD / tháng** | **$8.54 – $13.09 USD / tháng** |
| **QUY ĐỔI VNĐ** | **Tỷ giá quy đổi: 1 USD = 25.500 VNĐ** | **~249.000 – 364.000 VNĐ / tháng** | **~217.000 – 334.000 VNĐ / tháng** |
| **BÌNH QUÂN / SV** | **Quy mô 100 – 200 sinh viên** | **1.246 – 3.639 VNĐ / SV / tháng** | **1.088 – 2.177 VNĐ / SV / tháng** |

> **Đối chiếu Công cụ AWS Pricing Calculator và Biểu giá Niêm yết (Rate Card):**
> * Trên công cụ thực tế [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934), tổng chi phí cơ sở là **$4.96 USD / tháng** (bao gồm Rekognition $3.00, Transcribe Batch $1.50 theo đơn giá batch $0.0001/giây, S3 20GB $0.46, Lambda $0.00 Always Free).
> * Khi tính theo **Biểu giá Niêm yết tối đa (Worst-case Rate Card $0.024/phút cho Transcribe)** cộng thêm tầng suy luận Bedrock Nova Micro / Nova Pro ($3.56 – $4.04), S3 API requests và CloudWatch Logs, tổng ngân sách dài hạn nằm trong khoảng **$9.77 – $14.27 USD / tháng** (~1.200 – 2.500 VNĐ / sinh viên / tháng).
> * Hơn **82% chi phí hệ thống tập trung ở tầng AI Inference**. Toàn bộ hạ tầng Serverless (Lambda, SQS, S3, DynamoDB, CloudFront) chỉ tốn dưới **$1.50 USD / tháng**, tự động co về $0.00 USD khi ở trạng thái nghỉ.

---

## 5. Tiêu chí Đánh giá & Kiểm thử Hoàn thành Workshop (Acceptance Criteria)

Trước khi tiến hành nghiệm thu hoặc dọn dẹp tài nguyên (Cleanup), workshop được xác minh dựa trên các tiêu chí kỹ thuật thực tế sau:

| STT | Hạng mục kiểm thử | Phương thức xác minh | Kết quả kỹ thuật kỳ vọng | Dịch vụ AWS liên quan |
| :---: | :--- | :--- | :--- | :--- |
| **1** | Khởi tạo Hạ tầng IaC | `sam deploy` hoặc CloudFormation Console | Stack `edumedia-ai` (hoặc `agentsys`) ở trạng thái `CREATE_COMPLETE` hoặc `UPDATE_COMPLETE`, xuất đầy đủ Outputs (ApiEndpoint, S3 Bucket, TableName). | AWS SAM, CloudFormation |
| **2** | Kiểm tra Sức khỏe API | `GET /health` | Trả về HTTP 200 kèm trạng thái chi tiết của từng component (S3, DynamoDB, Rekognition, Bedrock). | API Gateway, Lambda |
| **3** | Phân tích Tư liệu Hình ảnh | Upload ảnh slide học tập lên S3 qua Web / API | SQS nhận thông điệp, kích hoạt Lambda Ingest gọi Rekognition đọc nhãn và trích xuất nguyên văn OCR; phân loại đúng danh mục và môn học. | S3, SQS, Rekognition, DynamoDB |
| **4** | Phân tích Tư liệu Âm thanh | Upload file audio bài giảng (`.m4a`, `.mp3`) | Kích hoạt tác vụ Speech Processing, theo dõi trạng thái qua EventBridge và lưu trữ bản ghi transcript vào S3 / DynamoDB. | S3, SQS, Transcribe, EventBridge |
| **5** | Truy vấn Tìm kiếm Thông minh | `POST /api/v1/media/agent/query` | Trả về Search Plan có cấu trúc (keywords, category, subject) và danh sách tệp khớp ngữ nghĩa kèm mốc bằng chứng. | Amazon Bedrock, DynamoDB GSI |
| **6** | Giám sát & Báo động Vận hành | Gửi tải hoặc tạo lỗi thử nghiệm | CloudWatch Logs ghi log JSON có cấu trúc (`MediaIngested`, `MediaIngestFailed`), 7 CloudWatch Alarms sẵn sàng bảo vệ hệ thống. | CloudWatch Logs, Alarms, SNS |
