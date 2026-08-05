---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Triển khai thật toàn bộ luồng đăng nhập Google + Cognito + TOTP lên EC2 production
* Phát hiện và xử lý dứt điểm các lỗi cấu hình/lỗi code phát sinh khi chạy thật
* Đảm bảo cả 2 vai trò (Member, Admin) đăng nhập được ổn định
* Lập các dịch vụ phụ trợ nhằm đảm bảo bảo trì lâu dài

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Đưa code Cognito/TOTP lên EC2, `composer install`, chạy migration DB | `20/07/2026` | 1 ngày |
| **2** | - Xử lý lỗi cấu hình OAuth: `invalid_scope`, thiếu Google trong danh sách Identity Provider của App Client | `21/07/2026` | 2 ngày |
| **3** | - Xử lý lỗi runtime: lỗi 500 khi tạo user mới từ Cognito (SQL bind sai tham số) | `23/07/2026` | 1 ngày |
| **4** | - Kiểm thử toàn bộ luồng: Member đăng nhập Google, Admin đăng nhập Google + TOTP, đăng nhập username/password cũ <br> - Gắn CloudWatch cho EC2 và RDS ( Notification là Gmail từ dịch vụ SNS ) để thông báo lưu lượng đột biến từ máy chủ| `24/07/2026` | 2 ngày |
| **5** | - Đồng bộ lại code đã sửa trực tiếp trên EC2 ngược về GitHub, dọn dẹp bảo mật (đổi mật khẩu RDS, reset Google Client Secret) | `26/07/2026` | 1 ngày |

### Kết quả đạt được tuần 7:

* Luồng đăng nhập Google cho cả Member và Admin chạy ổn định trên production, Admin bắt buộc phải qua bước TOTP mới vào được `/admin`.
* Xác định và vá được 3 lỗi chính:
  * **`invalid_scope`**: App Client thiếu tick scope `profile` trong OAuth Connect scopes.
  * **Lỗi 400 chung chung từ Cognito**: dùng AWS CLI (qua CloudShell) để đọc trực tiếp cấu hình App Client, phát hiện `SupportedIdentityProviders` chỉ có `COGNITO`, thiếu `Google` — sửa qua `update-user-pool-client`.
  * **Lỗi 500 khi tạo user mới**: hàm sinh username tự động gọi lồng 1 câu SQL SELECT ngay giữa lúc câu SQL INSERT đang bind tham số, làm lệch dữ liệu bind (do lớp `Database` dùng chung 1 prepared statement cho toàn app) — sửa bằng cách tính username trước khi bắt đầu bind.
* Học được cách dùng AWS CLI/CloudShell để debug trực tiếp thay vì chỉ dựa vào giao diện Console — hiệu quả hơn nhiều khi Console hiển thị lỗi không rõ ràng.

### Hạn chế:

* Vẫn phải debug thủ công qua log Apache (`error_log`) vì chưa bật CloudWatch Log streaming cho Cognito (thuộc gói Plus, tính phí) — chấp nhận đánh đổi cho giai đoạn học tập.
* Một vài lần deploy bị thiếu đồng bộ giữa EC2 và GitHub (quên push file, đặt sai tên file) — cho thấy rõ nhu cầu cần có CI/CD tự động ở các dự án sau.