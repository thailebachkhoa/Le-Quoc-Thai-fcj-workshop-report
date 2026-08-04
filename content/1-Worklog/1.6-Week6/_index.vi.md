---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Thay thế đăng nhập username/password truyền thống bằng đăng nhập Google, dùng Amazon Cognito làm trung gian xác thực
* Bắt buộc xác thực 2 lớp (TOTP - Google Authenticator) riêng cho tài khoản Admin
* Cập nhật lại code + database cho phù hợp với luồng xác thực mới

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Tạo Cognito User Pool + App Client, tạo Cognito Domain (Hosted UI) | `13/07/2026` | 1 ngày |
| **2** | - Tạo OAuth Client trên Google Cloud Console, thêm Google làm Identity Provider trong Cognito, cấu hình attribute mapping (email, name) | `14/07/2026` | 1 ngày |
| **3** | - Viết `Cognito.php` (đổi authorization code lấy token, verify chữ ký JWT qua JWKS) và `Totp.php` (sinh/verify mã TOTP theo chuẩn RFC 6238, không phụ thuộc thư viện ngoài) | `15/07/2026` | 2 ngày |
| **4** | - Sửa `AuthController`, `User` model: thêm luồng `google()`/`callback()`/`totp()`, đồng bộ vai trò Admin/Member từ Cognito Group vào cột `role` sẵn có | `17/07/2026` | 2 ngày |
| **5** | - Viết migration thêm cột `cognito_sub`, `totp_secret` vào bảng `users`; thêm view `auth/totp.php` (màn hình quét QR/nhập mã) | `19/07/2026` | 1 ngày |

### Kết quả đạt được tuần 6:

* Chốt được kiến trúc xác thực cuối cùng: User & Admin đều đăng nhập bằng nút "Đăng nhập với Google" duy nhất; PHP đọc Cognito Group trong token để phân biệt vai trò; riêng Admin bị chặn thêm 1 bước nhập mã TOTP trước khi được cấp session thật.
* Giữ lại được form đăng nhập username/password cũ song song (không xoá) để dễ rollback, đồng thời áp luôn quy tắc bắt buộc TOTP cho Admin ở cả 2 đường đăng nhập — không có đường tắt nào bỏ qua được lớp bảo vệ này.
* Toàn bộ code liên quan (Cognito, TOTP, migration DB) đã viết xong và sẵn sàng để triển khai lên EC2 thật ở tuần sau.

### Hạn chế:

* Phần cấu hình AWS Console/Google Cloud Console mới chỉ làm trên lý thuyết và sổ tay hướng dẫn, chưa test thật trên môi trường production — dự kiến sẽ phát sinh khá nhiều lỗi cấu hình khi triển khai thật (kiểm chứng ở Tuần 7).