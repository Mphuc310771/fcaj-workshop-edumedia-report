# EduMedia AI Production Demo & Evaluation Runbook

> **Mục tiêu:** Tài liệu thực chiến hướng dẫn giám khảo và kỹ sư vận hành triển khai hệ thống EduMedia AI (Backend Serverless + Frontend Web Gallery + Báo cáo FCJ), cấu hình CORS đa môi trường, và thực hiện kịch bản nghiệm thu 5 bước bằng `curl` độc lập cùng bộ tiêu chí đánh giá Web UI & CloudWatch Alarms.

---

## Mục lục
1. [Triển khai Backend Serverless (1-Click SAM Deploy)](#1-triển-khai-backend-serverless-1-click-sam-deploy)
2. [Triển khai Frontend Gallery Web (Vercel / CloudFront)](#2-triển-khai-frontend-gallery-web-vercel--cloudfront)
3. [Kịch bản 5 Bước Nghiệm thu Thực chiến (CLI / curl)](#3-kịch-bản-5-bước-nghiệm-thu-thực-chiến-cli--curl)
4. [Checklist Đánh giá Giao diện Gallery Web UI](#4-checklist-đánh-giá-giao-diện-gallery-web-ui)
5. [Kiểm tra Vận hành & 7 CloudWatch Alarms](#5-kiểm-tra-vận-hành--7-cloudwatch-alarms)
6. [Quy trình Dọn dẹp Tài nguyên (Zero-Cost Cleanup)](#6-quy-trình-dọn-dẹp-tài-nguyên-zero-cost-cleanup)

---

## 1. Triển khai Backend Serverless (1-Click SAM Deploy)

### Cách 1: Sử dụng Script Python Tự động (Khuyến nghị)
Hệ thống cung cấp script tự động đóng gói dependencies, tạo `samconfig.toml`, đồng bộ `.env` và cập nhật S3 bucket:

```bash
cd /media/mewmewangriii/NewVolume/Code/Thuc_Tap/agent_sys

# Cấu hình biến môi trường tối thiểu trong agent_sys/.env
# ADMIN_API_KEY=your-secret-api-key-2026
# FRONTEND_ORIGIN_URL=* (hoặc domain frontend thật của bạn)
# ALERT_EMAIL=your-email@example.com (tùy chọn)

python scripts/deploy_stack.py
```

### Cách 2: Lệnh SAM CLI trực tiếp
```bash
cd /media/mewmewangriii/NewVolume/Code/Thuc_Tap/agent_sys

# 1. Đóng gói mã nguồn Lambda vào thư mục pack/
rm -rf pack && mkdir pack
cp -r apps config pipeline pack/
cat << 'EOF' > pack/requirements.txt
fastapi>=0.115.0
mangum>=0.19.0
pydantic>=2.9.2
pydantic-settings>=2.5.2
boto3>=1.35.36
python-multipart>=0.0.12
slowapi>=0.1.9
anyio>=4.0.0
flashtext>=2.7
PyYAML>=6.0.1
Pillow>=10.0.0
EOF

# 2. Build SAM Template
sam build -t infra/template.yaml

# 3. Deploy CloudFormation Stack
sam deploy \
  --stack-name agentsys \
  --region us-east-1 \
  --capabilities CAPABILITY_IAM \
  --resolve-s3 \
  --parameter-overrides \
    ApiKey="your-secret-api-key-2026" \
    FrontendOriginUrl="*"
```

Sau khi hoàn tất, lưu lại các giá trị tại phần **Outputs**:
- `ApiEndpoint`: URL gọi HTTP API Gateway (Ví dụ: `https://a1b2c3d4.execute-api.us-east-1.amazonaws.com` - *Lưu ý: Không có hậu tố `/Prod`*).
- `MediaBucketName`: Tên S3 Media Bucket (Ví dụ: `agentsys-mediabucket-xxxx`).
- `MetadataTableName`: `AgentSysMediaMetadata`.
- `AlertsTopicArn`: SNS Topic cảnh báo CloudWatch.

---

## 2. Triển khai Frontend Gallery Web (Vercel / CloudFront)

Frontend `gallery_web` được phát triển bằng React + Vite + TailwindCSS.

### Bước 2.1: Cấu hình biến môi trường
Tạo file `agent_sys/gallery_web/.env.production` (hoặc cấu hình trong Dashboard Vercel):
```bash
# URL lấy từ ApiEndpoint output của SAM (không có trailing slash)
VITE_API_BASE_URL=https://a1b2c3d4.execute-api.us-east-1.amazonaws.com

# API Key quản trị (nếu để trống, người dùng có thể nhập trực tiếp trên modal Web UI)
VITE_API_KEY=your-secret-api-key-2026
```

### Bước 2.2: Build kiểm tra cục bộ
```bash
cd /media/mewmewangriii/NewVolume/Code/Thuc_Tap/agent_sys/gallery_web
npm install
npm run build
```

### Bước 2.3: Deploy lên Vercel (1-Click)
Repository đã tích hợp sẵn file cấu hình SPA `agent_sys/gallery_web/vercel.json` (chuyển tiếp mọi route về `index.html`):
```bash
cd /media/mewmewangriii/NewVolume/Code/Thuc_Tap/agent_sys/gallery_web
npx vercel --prod
```
*Khi được hỏi root directory, chọn `./` (trong thư mục `gallery_web`) và thiết lập các biến `VITE_API_BASE_URL`, `VITE_API_KEY` trong Vercel Project Settings.*

---

## 3. Kịch bản 5 Bước Nghiệm thu Thực chiến (CLI / curl)

> **Xác minh Bộ Kiểm Thử Tự Động (143/143 Tests Passed - 84% Coverage):**  
> Trước khi thực hiện các lệnh curl, giám khảo có thể chạy xác thực toàn bộ logic hệ thống Backend:
> ```bash
> pytest agent_sys/tests/ -q
> # Kết quả: 143 passed, 3 warnings in ~28s (Total Coverage: 84%)
> ```

Giám khảo và kỹ sư có thể kiểm thử toàn bộ tính năng và FinOps/Demo Guardrails trực tiếp từ terminal mà không cần mở trình duyệt:

```bash
# Thiết lập biến môi trường test
export API_BASE="https://a1b2c3d4.execute-api.us-east-1.amazonaws.com"
export API_KEY="your-secret-api-key-2026"
```

### Bước 1: Health Check Hệ thống
Kiểm tra khả năng sẵn sàng của HTTP API Gateway và Lambda Handler:
```bash
curl -s -X GET "${API_BASE}/health" | jq .
```
**Kết quả kỳ vọng (HTTP 200):**
```json
{
  "status": "ok",
  "provider": "aws",
  "services": {
    "dynamodb": "connected",
    "s3": "connected"
  }
}
```

---

### Bước 2: Xin Presigned URL & Kiểm tra Guardrails Tải lên
#### 2a. Yêu cầu tải lên file hợp lệ (Ảnh $\le$ 25MB):
```bash
curl -s -X POST "${API_BASE}/api/v1/media/upload-url" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{
    "filename": "demo_test.jpg",
    "content_type": "image/jpeg",
    "size_bytes": 1048576
  }' | jq .
```
**Kết quả kỳ vọng (HTTP 200):** Trả về `file_id`, `s3_key`, và cả 2 trường `upload_url` & `presigned_url`.

#### 2b. Kiểm tra FinOps Guardrail: Chặn file vượt quá 25MB (Payload Too Large):
```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" -X POST "${API_BASE}/api/v1/media/upload-url" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{
    "filename": "heavy_video.mp4",
    "content_type": "video/mp4",
    "size_bytes": 31457280
  }'
```
**Kết quả kỳ vọng:** `HTTP Status: 413` (*File exceeds maximum public demo size limit of 25MB*).

#### 2c. Kiểm tra MIME Guardrail: Chặn tệp không hợp lệ (Executable, Archive, PDF):
```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" -X POST "${API_BASE}/api/v1/media/upload-url" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{
    "filename": "exploit.exe",
    "content_type": "application/x-msdownload",
    "size_bytes": 50000
  }'
```
**Kết quả kỳ vọng:** `HTTP Status: 400` (*Unsupported media type. Only image, audio, and video are allowed*).

---

### Bước 3: Truy vấn Ngôn ngữ Tự nhiên qua Amazon Bedrock (Nova Lite + Titan V2)
Kiểm tra khả năng tìm kiếm ngữ nghĩa thông minh và tổng hợp câu trả lời bằng tiếng Việt:
```bash
curl -s -X POST "${API_BASE}/api/v1/media/agent/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{
    "query": "tìm bài giảng hoặc bảng viết về đạo hàm ma trận",
    "top_k": 3
  }' | jq .
```
*(Hệ thống hỗ trợ cả endpoint tương thích `/agent/query` và payload `{"user_query": "..."}`).*

**Kết quả kỳ vọng (HTTP 200):**
```json
{
  "query": "tìm bài giảng hoặc bảng viết về đạo hàm ma trận",
  "plan": {
    "intent": "search",
    "category": "image"
  },
  "answer": "Tìm thấy bài giảng phù hợp với nội dung bài toán đạo hàm ma trận...",
  "items": [ ... ]
}
```

---

### Bước 4: Kiểm tra Demo Guardrail: Chặn xóa dữ liệu Demo (HTTP 403 Forbidden)
Đảm bảo khách vãng lai hoặc bot phá hoại không thể xóa mất dữ liệu bài giảng mẫu:
```bash
# Thử xóa tệp mẫu sample_lecture_01
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" -X DELETE "${API_BASE}/api/v1/media/items/sample_lecture_01" \
  -H "X-API-Key: ${API_KEY}"
```
**Kết quả kỳ vọng:** `HTTP Status: 403` (*Demo items cannot be deleted to preserve public demo integrity*).

*Muốn xóa bắt buộc phải truyền header quản trị viên: `-H "x-admin-force: true"`.*

---

### Bước 5: Xóa File Người Dùng Tải Lên Bình Thường (HTTP 200 OK)
```bash
# Upload thử một file test sau đó lấy file_id để xóa
TEST_ID=$(curl -s -X POST "${API_BASE}/api/v1/media/upload-url" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{"filename":"test.jpg","content_type":"image/jpeg","size_bytes":1000}' | jq -r .file_id)

echo "Testing deletion for user item: $TEST_ID"
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" -X DELETE "${API_BASE}/api/v1/media/items/${TEST_ID}" \
  -H "X-API-Key: ${API_KEY}"
```
**Kết quả kỳ vọng:** `HTTP Status: 200` (Xóa thành công khỏi DynamoDB và S3).

---

## 4. Checklist Đánh giá Giao diện Gallery Web UI

Khi truy cập vào đường link Vercel / CloudFront của Frontend:

- [ ] **Tự động đăng nhập / API Key Modal:** Nếu file `.env.production` có `VITE_API_KEY`, ứng dụng tự động kết nối mà không bắt người dùng nhập thủ công. Nếu không có, xuất hiện modal cấu hình thân thiện.
- [ ] **Danh sách Media Card:** Hiển thị thẻ tệp đa phương tiện (Ảnh, Âm thanh, Video) với huy hiệu danh mục, ngày tải lên và định dạng.
- [ ] **Huy hiệu Bảo vệ Demo (Shield Badge):** Các file mẫu (`is_demo_protected: true`) có biểu tượng khiên bảo vệ màu xanh lam. Nút xóa (Trash) bị vô hiệu hóa kèm thông báo giải thích rõ ràng.
- [ ] **Kéo thả Tải lên (Drag & Drop Upload):** 
  - Kéo file `.jpg` $\le$ 25MB: Hiển thị thanh tiến trình tải lên thành công.
  - Kéo file `.pdf` hoặc file > 25MB: Báo lỗi màu đỏ ngay lập tức trên giao diện mà không tốn băng thông gọi API.
- [ ] **Tìm kiếm Tự nhiên (Bedrock Search):** Nhập câu hỏi tiếng Việt (ví dụ: *"Tìm bài giảng về thuật toán"*), xem AI suy luận kế hoạch (Query Plan), highlight từ khóa liên quan và hiển thị câu trả lời AI tổng hợp.
- [ ] **Chi tiết Tệp & S3-Offload Transcript:** Nhấp vào tệp âm thanh dài > 250KB, modal hiển thị đoạn xem trước (lưu tối đa 15.000 ký tự trên DynamoDB) và nút bấm *"Tải toàn văn Transcript (JSON)"* lấy trực tiếp từ S3 (`transcripts/raw/{file_id}.json`).

---

## 5. Kiểm tra Vận hành & 7 CloudWatch Alarms

Hệ thống được giám sát liên tục bởi 7 CloudWatch Alarms đồng bộ tới SNS Topic:

```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix "agentsys-" \
  --region us-east-1 \
  --query "MetricAlarms[*].{Name:AlarmName, State:StateValue, Metric:MetricName}" \
  --output table
```

| Tên Alarm | Ý nghĩa giám sát | Trạng thái chuẩn |
| :--- | :--- | :---: |
| **`agentsys-api-errors`** | Lỗi 5xx trên API Gateway / FastAPI | `OK` |
| **`agentsys-ingest-errors`** | Lỗi runtime trong Lambda Ingest | `OK` |
| **`agentsys-aggregator-errors`** | Lỗi runtime trong Lambda Aggregator | `OK` |
| **`agentsys-ingest-failed`** | Custom Metric `MediaIngestFailed` $\ge 1$ | `OK` |
| **`agentsys-ingest-dlq`** | Tin nhắn rơi vào Ingestion DLQ | `OK` |
| **`agentsys-aggregator-stream-dlq`**| Lỗi luồng DynamoDB Stream rơi vào DLQ | `OK` |
| **`agentsys-transcribe-dlq`** | Sự kiện Transcribe EventBridge rơi vào DLQ | `OK` |

---

## 6. Quy trình Dọn dẹp Tài nguyên (Zero-Cost Cleanup)

Sau khi ban giám khảo hoàn tất chấm thi và nghiệm thu, thực hiện dọn dẹp để đưa chi phí về **`$0.00 USD`**:

```bash
# 1. Làm rỗng S3 Media Bucket (Bắt buộc trước khi xóa Stack)
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name agentsys \
  --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='MediaBucketName'].OutputValue" \
  --output text)

aws s3 rm "s3://${BUCKET_NAME}" --recursive

# 2. Xóa CloudFormation Stack
sam delete --stack-name agentsys --region us-east-1 --no-prompts

# 3. Xác minh stack đã xóa hoàn toàn
aws cloudformation describe-stacks --stack-name agentsys --region us-east-1
# Kỳ vọng: 'Stack with id agentsys does not exist'
```
