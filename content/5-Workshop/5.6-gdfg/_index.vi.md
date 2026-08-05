---
title: "6. Thay thế Auth bằng Cognito + Google OAuth + TOTP"
weight: 6
date: 2026-08-05
draft: false
---

## 6.1. Quyết định kiến trúc

Thay vì tự viết toàn bộ logic đăng nhập/mật khẩu (rủi ro bảo mật cao hơn nếu tự làm sai), chuyển sang **Amazon Cognito** làm nhà cung cấp danh tính, dùng **Google** làm Identity Provider thật (đăng nhập bằng tài khoản Google, không phải mã OTP qua email).

Luồng xác thực:

```
User bấm "Đăng nhập Google"
    → Cognito Hosted UI
        → Chuyển hướng sang Google thật (accounts.google.com)
            → User đăng nhập bằng tài khoản Google
        → Google redirect về Cognito domain (/oauth2/idpresponse)
    → Cognito phát hành id_token (JWT), redirect về app (/auth/callback)
    → PHP verify JWT, đọc cognito:groups để biết Admin hay Member
    → Nếu là Admin: bắt buộc thêm bước xác thực TOTP trước khi vào /admin
```

**Lưu ý quan trọng**: Cognito **không cho phép** kết hợp MFA có sẵn của Cognito với đăng nhập passwordless trên cùng 1 tài khoản. Vì vậy 2FA cho Admin được **tự viết ở tầng ứng dụng** (không dùng tính năng MFA built-in của Cognito), độc lập hoàn toàn với luồng xác thực chính.

## 6.2. Chuẩn bị Google Cloud OAuth Client

1. Google Cloud Console → tạo Project (dùng tài khoản Google cá nhân, tránh vướng giới hạn quyền của tài khoản tổ chức).
2. APIs & Services → OAuth consent screen → cấu hình, sau đó **Publish app** (mặc định chế độ Testing chỉ cho tối đa 100 user thử nghiệm).
3. Credentials → Create OAuth Client ID → Web application → Authorized redirect URI: `https://<cognito-domain>.auth.<region>.amazoncognito.com/oauth2/idpresponse`.

## 6.3. Cấu hình Cognito

1. **User pools → Create user pool** → Application type: Traditional web application.
2. **Sign-in experience → Federated identity provider sign-in → Add identity provider → Google** → nhập Client ID/Secret từ bước trên, Authorized scopes: `profile email openid`.
3. **App integration → Domain** → tạo Cognito domain (Hosted UI).
4. **App clients → Create app client** → Hosted UI settings:
   - Allowed callback URLs: `https://<domain>/auth/callback`
   - Identity providers: tick cả **Google** (đây là bước dễ bị bỏ sót nhất — mặc định chỉ có "Cognito user pool directory")
   - OAuth grant types: Authorization code grant
   - OpenID Connect scopes: `email`, `openid`, `profile`

## 6.4. Migration database

```sql
ALTER TABLE users
    ADD COLUMN cognito_sub VARCHAR(100) NULL UNIQUE AFTER id,
    ADD COLUMN totp_secret VARCHAR(64) NULL AFTER role,
    MODIFY COLUMN password VARCHAR(255) NULL;
```

`password` cho phép NULL vì tài khoản đăng nhập qua Google không còn mật khẩu cục bộ.

## 6.5. Cấu trúc code PHP

- `app/Core/Cognito.php` — xử lý redirect OAuth, đổi authorization code lấy token, verify JWT bằng `firebase/php-jwt`.
- `app/Core/Totp.php` — sinh/xác thực mã TOTP 6 số, tự viết không phụ thuộc thư viện ngoài.
- `app/Views/auth/totp.php` — màn hình nhập mã Google Authenticator cho Admin.
- `.env` bổ sung: `COGNITO_REGION`, `COGNITO_USER_POOL_ID`, `COGNITO_APP_CLIENT_ID`, `COGNITO_APP_CLIENT_SECRET`, `COGNITO_DOMAIN`, `COGNITO_REDIRECT_URI`.

## 6.6. Tạo tài khoản Admin đầu tiên

Cognito Hosted UI chỉ cho phép tự đăng ký thành **Member** — không có form nào để tự nhận Admin (đúng nguyên tắc bảo mật). Việc "thăng cấp" Admin đầu tiên làm thủ công qua CLI:

```bash
aws cognito-idp admin-add-user-to-group \
  --user-pool-id <user-pool-id> \
  --username <email> \
  --group-name Admin
```

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| `composer install` chặn bởi cảnh báo bảo mật | Bản `firebase/php-jwt` cũ có lỗ hổng đã biết | Nâng lên phiên bản mới (v7+) |
| `invalid_scope` / `invalid_request` khi bấm đăng nhập | Thiếu tick scope `profile` trong App Client | Thêm `profile` vào OpenID Connect scopes |
| Lỗi 400 mơ hồ dù scope đã đúng | App Client chưa bật **Google** trong danh sách Identity providers được phép (`SupportedIdentityProviders` chỉ có `COGNITO`) | Vào Login pages → Edit → tick thêm Google vào Identity providers |
| Lỗi 500 khi xử lý callback | `Database` là singleton, gọi lồng 2 câu SQL cùng lúc làm lệch tham số bind | Tránh gọi truy vấn lồng nhau trên cùng 1 kết nối PDO singleton, tách thành 2 lệnh tuần tự |
| File `login.php` hiển thị bản cũ trên server dù đã sửa local | Quên `git push` file đó trước khi `git pull` trên EC2 | Kiểm tra `git status`/`git log` trước khi pull để chắc chắn đã push đủ |