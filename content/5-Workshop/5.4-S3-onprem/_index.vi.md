---
title: "4. Domain, HTTPS, Elastic IP"
weight: 4
date: 2026-08-05
draft: false
---

## 4.1. Gắn Elastic IP

Public IP mặc định của EC2 đổi mỗi lần Stop/Start — gây phiền vì phải sửa lại SSH/domain liên tục. Gắn địa chỉ cố định:

**EC2 → Network & Security → Elastic IPs → Allocate Elastic IP address** → **Actions → Associate Elastic IP address** → chọn đúng instance.

> Lưu ý về chi phí: từ 1/2/2024, AWS tính phí mọi địa chỉ IPv4 công khai (~$0.005/giờ), kể cả khi đang gắn với instance đang chạy. Free Tier EC2 bao gồm 750 giờ IPv4/tháng trong 12 tháng đầu — đủ miễn phí nếu chỉ dùng đúng 1 địa chỉ.

## 4.2. Domain miễn phí qua DuckDNS

Đăng ký tại [duckdns.org](https://www.duckdns.org), trỏ subdomain về Elastic IP vừa tạo. Cập nhật lại `ServerName` trong Apache Virtual Host cho khớp domain mới.

## 4.3. HTTPS qua Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d <subdomain>.duckdns.org
```

Chọn tùy chọn **redirect toàn bộ HTTP sang HTTPS** khi Certbot hỏi. Chứng chỉ tự động gia hạn, không cần thao tác thêm.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| Sau khi Stop/Start, không SSH vào được bằng IP cũ | Chưa gắn Elastic IP, Public IP đã đổi | Kiểm tra lại Public IP mới trên EC2 Console, hoặc gắn Elastic IP để tránh lặp lại |
| Certbot báo lỗi không cấp được chứng chỉ | Domain chưa thực sự trỏ đúng IP, hoặc cổng 80 chưa mở trong Security Group | Kiểm tra `ping <domain>` trả về đúng IP, xác nhận Security Group cho phép HTTP/HTTPS |