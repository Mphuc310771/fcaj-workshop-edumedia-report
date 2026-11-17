---
title: "Tuần 10"
date: 2026-10-23
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

# Tuần 10: Phát Triển Giao Diện Web Gallery & Tích Hợp Client
*Thời gian thực hiện: 19/10/2026 – 23/10/2026*

### 1. Mục tiêu tuần 10:
* Khởi tạo và thiết kế giao diện Web Gallery hiện đại (React, Tailwind CSS, Vite) kết nối trực tiếp với backend REST API của EduMedia AI.
* Hiện thực hóa tính năng Media Player tương tác: Click-to-Seek đồng bộ trực tiếp theo timestamp câu thoại trong bản ghi transcript bài giảng.
* Thiết lập các chốt chặn kiểm soát và bảo vệ dữ liệu hệ thống (System Guardrails): cơ chế chặn xóa tệp gốc được bảo vệ (HTTP 403), giới hạn dung lượng tải lên (<= 25MB) và cấu hình chính sách CORS trên API Gateway.
* Tích hợp bộ lọc tìm kiếm theo danh mục (category chips), môn học và kết nối endpoint tra cứu thông minh `POST /api/v1/media/agent/query`.

### 2. Các công việc triển khai trong tuần:
| Thứ | Công việc cụ thể | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | - Khởi tạo dự án Web Frontend với Vite, React và Tailwind CSS.<br>- Thiết kế layout dạng lưới thẻ media trực quan và responsive trên các thiết bị. | 19/10/2026 | 19/10/2026 | [Vite & React Guide](https://vitejs.dev/) |
| **3** | - Tích hợp API Gateway Client, kết nối endpoint GET /api/v1/media để tải danh mục.<br>- Hiển thị thumbnail ảnh, thông tin trích xuất nhãn và văn bản OCR từ Rekognition. | 20/10/2026 | 20/10/2026 | [Amazon API Gateway Guide](https://docs.aws.amazon.com/apigateway/) |
| **4** | - Xây dựng tính năng Click-to-Seek Media Player.<br>- Bấm vào từng đoạn transcript sẽ tự động nhảy đến đúng mốc thời gian âm thanh/video phát trong bài giảng. | 21/10/2026 | 21/10/2026 | [HTML5 Media Audio and Video API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement) |
| **5** | - Thiết lập các cơ chế kiểm soát và bảo vệ hệ thống (System Guardrails & Input Validation).<br>- Ràng buộc cờ bảo vệ tài nguyên (chặn xóa dữ liệu gốc trả về HTTP 403), giới hạn dung lượng tải lên (<= 25MB) và cấu hình CORS FrontendOriginUrl. | 22/10/2026 | 22/10/2026 | [API Gateway CORS Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html) |
| **6** | - Kiểm thử trải nghiệm người dùng trên máy cục bộ, tối ưu tốc độ render component.<br>- Kết nối giao diện tìm kiếm với API Bedrock Agent Query trả về Search Plan trực quan. | 23/10/2026 | 23/10/2026 | [React Hooks Documentation](https://react.dev/) |

### 3. Kết quả đạt được tuần 10:
* Hoàn thiện ứng dụng Web Gallery hoàn chỉnh kết nối mượt mà với các API Serverless của EduMedia AI.
* Tính năng Click-to-Seek media player đồng bộ mốc thời gian bài giảng hoạt động chính xác, nâng cao trải nghiệm tra cứu tư liệu học tập.
* Các chốt chặn bảo vệ và kiểm duyệt input giúp hệ thống ngăn chặn hiệu quả thao tác xóa ngoài ý muốn và chống tràn tải dữ liệu lớn.
* Giao diện tìm kiếm thông minh hiển thị kết quả phân loại môn học và trích dẫn bằng chứng rõ ràng.

### 4. Tài liệu tham khảo:
* [Vite Getting Started Guide](https://vitejs.dev/guide/)
* [Tailwind CSS Documentation](https://tailwindcss.com/docs)
* [Amazon API Gateway Enabling CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
* [HTML5 Media Audio and Video DOM Reference](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
