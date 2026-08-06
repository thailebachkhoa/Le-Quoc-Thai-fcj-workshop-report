---
title: "Amazon Cognito, giải thích cho đàng hoàng: User Pool, Hosted UI, Federation, và trong token thật sự có gì"
date: 2026-08-04
draft: false
tags: ["aws", "cognito", "identity", "authentication"]
description: "Giải thích từ gốc về Amazon Cognito — nó thực chất là gì, 2 nửa rất khác nhau bên trong nó, đăng nhập liên kết hoạt động ra sao phía sau hậu trường, và trong JWT nó trả về thật sự chứa gì."
---

Amazon Cognito là kiểu dịch vụ AWS bị dùng rất nhiều nhưng hiểu rất mơ hồ. Đa số hướng dẫn chỉ chỉ bạn bấm nút nào để có đăng nhập Google chạy được, mà không giải thích Cognito thực sự đang làm gì ở giữa. Bài này là phần giải thích lẽ ra nên đọc trước khi bắt tay xây dựng xác thực cho dự án Plantify Co, để đỡ tốn thời gian nhầm lẫn.

## Trước tiên: Cognito là 2 dịch vụ khác nhau đội chung 1 cái tên

Đây là nguồn nhầm lẫn phổ biến nhất. "Amazon Cognito" thực chất gộp 2 sản phẩm giải quyết 2 bài toán khác hẳn nhau:

- **User Pools** — dịch vụ quản lý danh mục user và xác thực. Trả lời câu *"đây là ai, và họ chứng minh điều đó bằng cách nào?"*. Nó có thể tự lưu user, hoặc làm trung gian lấy danh tính từ nhà cung cấp ngoài (Google, Facebook, SAML/OIDC doanh nghiệp). Đây là phần lo màn hình đăng nhập, mật khẩu, MFA, và đăng nhập liên kết.
- **Identity Pools** — cách để cấp **credential AWS tạm thời** cho user của app bạn, để app mobile/web gọi thẳng dịch vụ AWS (như S3) mà không cần backend làm trung gian. Trả lời câu *"giờ đã biết đây là ai rồi, thì họ được cấp quyền AWS gì?"*.

Rất nhiều dự án — kể cả Plantify Co — chỉ cần đúng **User Pools**. Nếu bạn không cho trình duyệt của user gọi thẳng API AWS, có thể bỏ qua Identity Pools hoàn toàn, và phần lớn tiếng xấu "Cognito khó hiểu" cũng biến mất theo.

## Bên trong 1 User Pool: các thành phần chính

- **User Pool** — chính là "danh bạ" user. Lưu bản ghi user, thuộc tính của họ (email, tên...), và cấu hình như chính sách mật khẩu hay yêu cầu MFA.
- **App Client** — đại diện cho *1 ứng dụng* được phép xác thực với pool đó. 1 User Pool có thể có nhiều App Client (ví dụ 1 web app và 1 mobile app), mỗi cái có riêng callback URL được phép, OAuth scope, và danh sách Identity Provider được bật. Đây là lý do vì sao quên bật Identity Provider ở cấp *App Client* — dù đã thêm nó vào User Pool rồi — là lỗi cấu hình cực kỳ hay gặp: cấp pool và cấp client được cấu hình **độc lập** với nhau.
- **Domain (Hosted UI)** — 1 trang đăng nhập dựng sẵn do AWS host tại `<prefix>.auth.<region>.amazoncognito.com`, để bạn không phải tự xây form đăng nhập chỉ để bắt đầu luồng OAuth2.
- **Identity Providers** — dịch vụ ngoài (Google, Facebook, 1 nhà cung cấp SSO...) mà Cognito có thể uỷ quyền việc xác thực, thay vì tự quản lý mật khẩu.
- **Groups** — cách đơn giản để gắn vai trò cho user (ví dụ `Admin`, `Member`); việc thuộc group nào sẽ hiện thẳng trong token phát hành ra.

## Luồng đăng nhập liên kết thật sự diễn ra thế nào

Khi user đăng nhập qua Google thông qua Cognito, có 4 bên tham gia, và cần chính xác ai nói chuyện với ai:

1. **App của bạn → Cognito**: redirect trình duyệt sang Hosted UI của Cognito, có thể kèm `identity_provider=Google` để nhảy thẳng sang Google, bỏ qua màn chọn của Cognito.
2. **Cognito → Google**: redirect trình duyệt tiếp, lần này sang đúng màn hình cấp quyền thật của Google. App của bạn hoàn toàn không tham gia bước này.
3. **Google → Cognito**: sau khi user đồng ý, Google gửi 1 authorization code về — nhưng về đúng redirect URI của *Cognito*, không phải của app bạn. Cognito sau đó tự gọi ngầm (server-to-server) sang endpoint token của Google để đổi code đó lấy token của Google, rồi tạo hoặc khớp 1 bản ghi user trong User Pool.
4. **Cognito → app của bạn**: redirect trình duyệt lần cuối, về đúng callback URL của *app bạn*, kèm 1 authorization code do Cognito phát hành. App bạn đổi code này lấy token của chính Cognito.

Điểm mấu chốt lộ ra ở đây: ứng dụng của bạn **không bao giờ** thấy được token của Google hay mật khẩu Google của user, tại bất kỳ thời điểm nào. Nó chỉ nhận được token **do Cognito phát hành** — đây chính xác là lý do vì sao verify đúng token đó lại quan trọng đến vậy: app bạn đang tin lời của Cognito, không phải tin trực tiếp Google.

## Trong token thật sự có gì

Cognito phát hành JSON Web Token (JWT) — 1 chuỗi 3 phần đã ký (`header.payload.signature`). Phần payload là tập hợp các claim, và với đăng nhập liên kết thường có dạng:

```json
{
  "sub": "a1b2c3d4-...",
  "iss": "https://cognito-idp.<region>.amazonaws.com/<user-pool-id>",
  "aud": "<app-client-id>",
  "token_use": "id",
  "email": "user@gmail.com",
  "name": "User Name",
  "cognito:groups": ["Admin"],
  "exp": 1735689600
}
```

Verify đúng cách phải kiểm tra nhiều hơn là "chuỗi này có decode ra JSON không" — cần xác nhận: **chữ ký** hợp lệ theo public key của Cognito (JWKS), `iss` khớp đúng User Pool của bạn, `aud` khớp đúng App Client của bạn, và `token_use` đúng loại token mong đợi (token `id` và `access` phục vụ mục đích khác nhau, không nên dùng lẫn lộn). Bỏ qua bất kỳ bước nào trong số này nghĩa là app bạn có thể chấp nhận 1 token bị giả mạo cho *app client khác* hoặc *pool khác* — decode JWT mà không verify về bản chất chẳng khác gì không kiểm tra xác thực gì cả.

## Cognito thật sự mạnh ở đâu, và không hợp ở đâu

**Hợp**: chuẩn hoá OAuth2/OIDC để không phải tự viết riêng cho từng nhà cung cấp, tập trung quản lý user/group ở 1 chỗ, và có sẵn Hosted UI nên ngay ngày đầu đã có màn đăng nhập chạy được.

**Không hợp**: bất kỳ logic nào cần áp dụng riêng theo từng user/nhóm *ngay trong* bước xác thực — ví dụ MFA của Cognito áp dụng ở cấp User Pool, không theo từng nhóm, và không ghép trơn tru với đăng nhập liên kết. Đọc thêm [Blog 1](../3.1-Blog1/) nếu muốn xem cách Plantify Co dùng IAM Role thay vì Cognito Identity Pools để cho EC2 nói chuyện với S3 — 1 ví dụ tốt về việc chọn đúng công cụ định danh AWS cho đúng tầng của hệ thống, thay vì dùng 1 công cụ cho mọi việc.

## Tổng kết

Cognito bớt bí ẩn hẳn khi ngừng coi nó là 1 hộp đen duy nhất, mà coi nó là: 1 danh bạ user (User Pools) có thể uỷ quyền cho nhà cung cấp ngoài, bọc trong 1 luồng OAuth2 chuẩn, phát hành ra JWT mà app bạn có trách nhiệm tự verify cho đúng. Phần lớn nhầm lẫn về Cognito — kể cả không ít lần debug trong chính dự án này — đều bắt nguồn từ việc gộp lẫn các mảnh đó lại với nhau thay vì suy luận từng mảnh một cách tách bạch.