---
title: "Tuần 8"
date: 2026-10-09
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Tuần 8: Xây Dựng Database Metadata & Tối Ưu DynamoDB GSI
*Thời gian thực hiện: 05/10/2026 – 09/10/2026*

### 1. Mục tiêu tuần 8:
* Thiết kế lược đồ cơ sở dữ liệu NoSQL trên Amazon DynamoDB (AgentSysMediaMetadata) với khóa chính đơn file_id.
* Phát hiện và giải quyết triệt để vấn đề chi phí và độ trễ khi sử dụng Scan kèm FilterExpression để lọc tệp theo danh mục (category).
* Triển khai Global Secondary Index (category-index) chuyển đổi độ phức tạp từ quét toàn bảng O(N) sang truy vấn phân vùng O(1).
* Xây dựng cơ chế S3-Offloading tự động chuyển toàn văn transcript > 250KB sang S3 để triệt tiêu nguy cơ vượt ngưỡng 400KB của DynamoDB.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Khởi tạo bảng DynamoDB AgentSysMediaMetadata trên SAM template.<br>- Cấu hình Partition Key file_id (String) và thiết lập chế độ On-Demand (Pay-Per-Request). | 05/10/2026 | 05/10/2026 | [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/) |
| **3** | - Thử nghiệm API lấy danh sách file theo danh mục.<br>- Phân tích hiện tượng Scan + FilterExpression bị trừ RCU toàn bộ bảng và gây lỗi Empty Page Pagination. | 06/10/2026 | 06/10/2026 | [DynamoDB Scan vs Query](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html) |
| **4** | - Thêm Global Secondary Index (GSI) category-index (PK: category, SK: created_at, Projection: INCLUDE) vào infra/template.yaml. | 07/10/2026 | 07/10/2026 | [DynamoDB GSI Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html) |
| **5** | - Viết cơ chế S3-Offloading trong worker.<br>- Tự động đẩy transcript thô vượt quá 250KB sang s3://MediaBucket/transcripts/raw/{file_id}.json chống tràn ngưỡng 400KB. | 08/10/2026 | 08/10/2026 | [DynamoDB Item Sizes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html#limits-items) |
| **6** | - Đo đếm và so sánh số liệu hiệu năng thực tế.<br>- Ghi nhận GSI giảm 99.98% RCU tiêu thụ và giảm thời gian phản hồi từ hàng giây xuống dưới 30ms. | 09/10/2026 | 09/10/2026 | [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html) |

### 3. Kết quả đạt được tuần 8:
* Khởi tạo thành công bảng DynamoDB AgentSysMediaMetadata với kiến trúc NoSQL chuẩn hóa.
* Thay thế hoàn toàn thao tác Scan bằng Query trên GSI category-index, cắt giảm 99.98% chi phí đọc RCU.
* Triệt tiêu hiện tượng Empty Page Pagination, giảm độ trễ phản hồi của API lọc từ vài giây xuống dưới 30ms.
* Triển khai cơ chế S3-Offloading bảo vệ bảng an toàn trước giới hạn kích thước tối đa 400KB/item của DynamoDB.

### 4. Tài liệu tham khảo:
* [DynamoDB Secondary Indexes Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)
* [DynamoDB Best Practices for Querying and Scanning](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html)
* [Handling Large Items and Attributes in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-use-s3-too.html)
