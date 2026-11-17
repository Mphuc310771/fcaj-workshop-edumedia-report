---
title: "Chuẩn bị"
date: 2026-10-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# 5.2. Chuẩn Bị Môi Trường, AWS CLI, SAM, IAM Least-Privilege & Bật Amazon Bedrock Model Access

---

## 1. Yêu cầu tiên quyết & Cài đặt công cụ

Trước khi bắt đầu, bạn cần chuẩn bị các công cụ sau trên máy tính cá nhân:

1. **Tài khoản AWS (AWS Account)** có quyền quản trị viên (AdministratorAccess) hoặc quyền khởi tạo IAM Role, S3, DynamoDB, Lambda, SQS, API Gateway, CloudWatch.
2. **AWS CLI v2**: Cài đặt và cấu hình thông qua aws configure.
3. **AWS SAM CLI**: Công cụ đóng gói và triển khai Serverless Application Model.
4. **Python 3.12** & **Node.js 20+** (để chạy Web UI).

### Xác minh cấu hình AWS CLI:
```bash
aws sts get-caller-identity
aws configure get region
```
*Đảm bảo kết quả trả về đúng User IAM của bạn và Region là **us-east-1** (N. Virginia).*

---

## 2. Thiết lập Ngân sách Chống Phát sinh Chi phí (AWS Budgets)

Để kiểm soát chặt chẽ chi phí và phòng ngừa phát sinh cước ngoài ý muốn:
1. Mở **AWS Management Console** → Tìm dịch vụ **AWS Budgets**.
2. Chọn **Create budget** → **Cost budget**.
3. Đặt tên: My Zero-Spend Budget → Đặt hạn mức: **$1.00 USD / tháng** (ngân sách Zero-Spend bảo vệ tài khoản).
4. Cấu hình gửi cảnh báo Email khi chi phí thực tế chạm hạn mức để luôn duy trì mức chi phí $0.00 USD.

---

## 3. Quản Lý Quyền Truy Cập Model Trên Amazon Bedrock (Cập Nhật Kích Hoạt Tự Động)

Theo chính sách mới nhất của AWS Bedrock, quy trình cấp quyền truy cập mô hình nền tảng (Foundation Models) đã được tối ưu hóa:

1. **Thông Báo Miễn Trừ Cấp Quyền Thủ Công (Retired Model Access Page):**
   * Đối với các Serverless Foundation Models của Amazon (như **Amazon Titan Text Embeddings V2** và **Amazon Nova Lite/Micro/Pro**), AWS đã chính thức chuyển đổi sang cơ chế **kích hoạt tự động (automatically enabled on-demand)**.
   * Người dùng không còn cần phải nộp form yêu cầu phê duyệt hoặc ấn nút "Modify model access" thủ công như các phiên bản cũ. Các mô hình này sẵn sàng sử dụng ngay lập tức khi tài khoản thực hiện API call đầu tiên (`InvokeModel` hoặc `Converse API`) trong Region thương mại (như `us-east-1`).

![Thông báo chính thức từ AWS Bedrock: Trang Model Access đã được tinh gọn và kích hoạt tự động theo nhu cầu](/images/5-Workshop/evidence/bedrock-model-access.png)

2. **Quy tắc quản trị bảo mật IAM:**
   * Thay vì rào cản thủ công trên Console, việc kiểm soát quyền gọi model hiện được quản trị chặt chẽ bằng **IAM Policies** gắn vào Lambda Execution Role (`bedrock:InvokeModel`) và Service Control Policies (SCP).
   * Điều này giúp kiến trúc hệ thống chuẩn bị hạ tầng Serverless nhanh chóng, hoàn toàn tự động hoá thông qua Infrastructure as Code (SAM).

{{% notice tip %}}
**Lưu ý:** Đối với một số mô hình của bên thứ ba (như Anthropic Claude thông qua AWS Marketplace), tài khoản có thể được yêu cầu khai báo use-case thông tin sử dụng trong lần gọi API đầu tiên.
{{% /notice %}}

---

## 4. Nguyên Tắc Phân Quyền Tối Thiểu (IAM Least-Privilege Roles)

Trong file SAM Template (infra/template.yaml), hệ thống tự động khởi tạo 3 Execution Role độc lập với đặc quyền tối thiểu:

1. **IngestFunctionRole** (agentsys-ingest):
   * Chỉ có quyền s3:GetObject trên bucket uploads/\*.
   * Quyền rekognition:DetectLabels và rekognition:DetectText.
   * Quyền transcribe:StartTranscriptionJob và transcribe:GetTranscriptionJob.
   * Quyền dynamodb:PutItem, dynamodb:UpdateItem trên bảng AgentSysMediaMetadata.
   * Quyền tiêu thụ tin nhắn từ agentsys-ingest-queue.
2. **AggregatorFunctionRole** (agentsys-aggregator):
   * Tiêu thụ sự kiện từ DynamoDB Stream (GetRecords, GetShardIterator, DescribeStream).
   * Đọc chunks metadata (BatchGetItem, GetItem, UpdateItem) trên bảng AgentSysMediaMetadata.
   * Quyền bedrock:InvokeModel trên amazon.nova-lite-v1:0 để tổng hợp transcript và OCR.
   * Quyền gửi tin nhắn tới AggregatorStreamDLQ.
   * Quyền phát metric tới CloudWatch namespace AgentSys.
3. **ApiFunctionRole** (agentsys-api):
   * Quyền tạo s3:PutObject Presigned URL.
   * Quyền đọc dynamodb:GetItem, dynamodb:Query (bao gồm GSI category-index), và dynamodb:DeleteItem.
   * Quyền bedrock:InvokeModel trên các foundation models được cấp phép (Amazon Nova Lite, Titan Embeddings).

---

## 5. Chạy Kiểm Thử Tự Động Trước Khi Triển Khai (Unit Testing)

Thực thi bộ kiểm thử unit test cục bộ với Pytest để xác minh tính đúng đắn của các module chính (pipeline ingest, xác thực API, và xử lý Bedrock) trước khi deploy lên AWS:

```bash
# 1. Clone repository mã nguồn dự án EduMedia AI
git clone https://github.com/Mphuc310771/edumedia-ai.git
cd edumedia-ai

# 2. Khởi tạo và kích hoạt môi trường ảo Python
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

# 3. Cài đặt thư viện phụ thuộc và chạy 143 unit test cases
pip install -r requirements.txt
pytest -v
```
*Kỳ vọng: Các test cases vượt qua thành công, xác nhận logic xử lý và phân quyền hoạt động ổn định.*

---

## 6. Đóng Gói & Triển Khai Bằng AWS SAM (Build & Deploy)

Từ thư mục dự án `edumedia-ai`:

```bash
sam build -t infra/template.yaml
sam deploy --guided
```

### Các thông số nhập trong SAM Guided Deploy:
| Tham số | Giá trị đề xuất | Ý nghĩa |
| :--- | :--- | :--- |
| **Stack Name** | edumedia-ai | Tên CloudFormation Stack (hoặc agentsys) |
| **AWS Region** | us-east-1 | Khu vực triển khai hỗ trợ đầy đủ Bedrock |
| **Parameter ApiKey** | dev-local-api-key | Khóa bảo vệ API (Header X-API-Key) |
| **Parameter FrontendOriginUrl** | \* (hoặc URL Vercel/CloudFront) | Domain được cấp phép CORS |
| **Confirm changes before deploy** | Y | Xác nhận tài nguyên trước khi tạo |
| **Allow SAM CLI IAM role creation** | Y | Cho phép SAM tự động tạo IAM Roles |

Sau khi triển khai hoàn tất, lưu lại các giá trị trong bảng **Outputs**:
* ApiEndpoint: URL gọi HTTP API Gateway (ví dụ: https://xxxx.execute-api.us-east-1.amazonaws.com - *lưu ý: HTTP API sử dụng $default stage nên không có hậu tố /Prod*)
* MediaBucketName: Tên S3 Bucket lưu trữ media
* MetadataTableName: AgentSysMediaMetadata
