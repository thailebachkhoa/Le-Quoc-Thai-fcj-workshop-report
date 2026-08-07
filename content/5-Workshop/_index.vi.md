---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Overview

Plantify Co là một ứng dụng web bán cây cảnh, viết bằng PHP thuần theo mô hình MVC tự triển khai, dùng MySQL làm cơ sở dữ liệu. Workshop này ghi lại toàn bộ quá trình đưa ứng dụng có sẵn (đang chạy local) lên hạ tầng AWS thật, đúng theo thứ tự đã thực hiện — bao gồm cả những chỗ làm sai, phải quay lại sửa, hoặc đổi hướng giữa chừng.

Mục tiêu của workshop:

- Triển khai một ứng dụng PHP truyền thống lên AWS theo mô hình 2 tầng (EC2 + RDS), không dùng serverless, phù hợp với ứng dụng đã có sẵn codebase.
- Áp dụng đầy đủ 5+ dịch vụ AWS, mỗi dịch vụ có vai trò rõ ràng, không dùng cho có.
- Đảm bảo 3 trụ cột: **bảo mật** (least privilege, HTTPS, CSRF), **khả năng giám sát** (CloudWatch, SNS), **tối ưu chi phí** (tự động Start/Stop tài nguyên).
- Thay thế hệ thống đăng nhập tự viết bằng **Amazon Cognito + Google OAuth**, có 2FA bắt buộc cho Admin.

Toàn bộ mã nguồn: [github.com/thailebachkhoa/FCAJ-Intern-Project](https://github.com/thailebachkhoa/FCAJ-Intern-Project)

Website demo: chạy qua domain DuckDNS, HTTPS bật qua Let's Encrypt.

## Prerequisite

Trước khi bắt đầu, cần chuẩn bị:

| Mục | Ghi chú |
|---|---|
| Tài khoản AWS | Còn trong Free Tier (12 tháng đầu) để không phát sinh chi phí ở hầu hết các bước |
| Region | `ap-southeast-1` (Singapore) — dùng xuyên suốt cho mọi dịch vụ để tránh độ trễ và chi phí truyền dữ liệu xuyên vùng |
| SSH client | Terminal có sẵn OpenSSH (WSL, Git Bash, PowerShell đều dùng được) |
| Git | Để clone code và đồng bộ giữa GitHub ↔ EC2 |
| Tài khoản Google Cloud | Để tạo OAuth Client cho luồng "Đăng nhập bằng Google" qua Cognito |
| Domain miễn phí | DuckDNS hoặc tương đương, để đủ điều kiện cấp SSL (Let's Encrypt không cấp cho địa chỉ IP thô) |
| Kiến thức nền | HTML/CSS/PHP cơ bản, khái niệm MVC, khái niệm VPC/Security Group ở mức sơ bộ (workshop sẽ giải thích khi cần) |

Không cần biết trước: Terraform/CDK (workshop này làm tay qua Console để dễ hiểu từng bước), Docker, Kubernetes.

## Mô tả kiến trúc

### Sơ đồ hạ tầng AWS


![Approve images](Y.jpg "Approve images")


### Bảng dịch vụ và vai trò

| Dịch vụ | Vai trò |
|---|---|
| Amazon EC2 | Server chạy Apache + PHP 8.5 + ffmpeg |
| Amazon RDS (MySQL) | Database, không public |
| Amazon S3 | Lưu bản sao lưu database |
| AWS IAM | Role + Policy Least Privilege cho EC2 và Lambda |
| Amazon CloudWatch | Giám sát CPU (EC2), dung lượng ổ đĩa (RDS) |
| Amazon SNS | Gửi email cảnh báo |
| AWS Lambda | Logic Start/Stop tài nguyên tự động |
| Amazon EventBridge Scheduler | Lên lịch chạy Lambda theo giờ |
| Amazon Cognito | User Pool, Hosted UI, quản lý Group Admin/Member |
| Google Cloud OAuth | Identity Provider xác thực tài khoản Google thật |

### Kiến trúc code (MVC)

```
Trình duyệt → index.php (router) → Controller (Auth + Csrf check)
                                        │
                            ┌───────────┴───────────┐
                            ▼                       ▼
                        Model (PDO)              View (render HTML)
                            │                       ▲
                            ▼                       │
                        RDS MySQL ──────────────────┘
```

## Các bước thực hành

Chia thành 7 phần theo đúng trình tự đã thực hiện — mỗi phần là 1 trang riêng, có thể đọc tuần tự hoặc nhảy tới phần cần tham khảo:

1. **Rà soát và vá bảo mật code trước khi lên cloud**
2. **Dựng hạ tầng cơ bản: EC2 + RDS**
3. **Mở rộng dịch vụ: S3, IAM, CloudWatch**
4. **Domain, HTTPS, Elastic IP**
5. **Tự động hoá tiết kiệm chi phí: Lambda + EventBridge**
6. **Thay thế Auth bằng Cognito + Google OAuth + TOTP**
7. **Vá lỗi UI mobile và dọn dẹp code thừa**
8. **Dọn dẹp**
Mỗi phần đều có mục **"Lỗi thường gặp"** ghi lại nguyên văn lỗi thật đã xảy ra trong quá trình làm, để người đọc theo sau không mất thời gian mò lại từ đầu.