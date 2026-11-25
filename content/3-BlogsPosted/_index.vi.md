---
title: "Các bài blogs đã đăng"
date: 2026-11-06
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Trong suốt quá trình thực tập tại chương trình First Cloud AI Journey (FCAJ) 2026 và phát triển dự án EduMedia AI, sinh viên đã nghiên cứu sâu, giải quyết các bài toán kỹ thuật phức tạp và chia sẻ các bài viết chuyên sâu lên cộng đồng [AWS Study Group (Facebook)](https://www.facebook.com/groups/awsstudygroupfcj):

---

### [Blog 1 - Xử lý Media Serverless: Tại sao mình không để Amazon S3 trigger trực tiếp Lambda?](3.1-blog1/)
* **Trạng thái công bố:** Đã duyệt & công khai — [Xem bài đăng trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2328280037936955/)  
Phân tích 3 cạm bẫy chí mạng của kiến trúc Push-based (S3 gọi trực tiếp Lambda): hàng đợi ngầm là hộp đen không đo được Queue Depth, khủng hoảng Concurrency kéo sập downstream và drop event thầm lặng khi retry kiệt. Bài viết trình bày giải pháp chuyển đổi sang mô hình Kéo (Pull-based) qua Amazon SQS + DLQ với cơ chế Claim Check, `ScalingConfig: MaximumConcurrency = 5` và `ReportBatchItemFailures`.

---

### [Blog 2 - Tối ưu truy vấn Amazon DynamoDB: Hạn chế của FilterExpression và giải pháp Global Secondary Index](3.2-blog2/)
* **Trạng thái công bố:** Đã gửi bài lên AWS Study Group VN *(Đang chờ quản trị viên phê duyệt — đã đính kèm ảnh chụp minh chứng bên trong)*  
Bài viết phân tích chuyên sâu cơ chế tầng vật lý của Amazon DynamoDB: tại sao FilterExpression trong thao tác Scan tiêu tốn 100% RCU trước khi lọc và gây hiện tượng Empty Page Pagination trên AWS Lambda, đồng thời đề xuất giải pháp định tuyến O(1) bằng Global Secondary Index (category-index) giúp giảm 99.98% RCU và hạ độ trễ từ vài giây xuống dưới 15ms.

---

### [Blog 3 - Tối ưu thời gian build Docker trên AWS CodeBuild với Amazon ECR Remote Cache](3.3-blog3/)
* **Trạng thái công bố:** Đã gửi bài lên AWS Study Group VN *(Đang chờ quản trị viên phê duyệt — đã đính kèm ảnh chụp minh chứng bên trong)*  
Phân tích hạn chế của cơ chế bộ nhớ đệm cục bộ `LOCAL_DOCKER_LAYER_CACHE` trên AWS CodeBuild (bị thu hồi khi các lần chạy cách nhau nhiều giờ/ngày, gây cold build ~10 phút). Đề xuất mô hình Remote Cache với **Docker Buildx (`containerd` Image Store)** kết hợp `--cache-from` / `--cache-to` (`image-manifest=true`) lưu trữ dài hạn trên **Amazon ECR**, giúp hạ thời gian build xuống ~6 phút và giảm tới 25% thời lượng build cho các pipeline multi-stage.