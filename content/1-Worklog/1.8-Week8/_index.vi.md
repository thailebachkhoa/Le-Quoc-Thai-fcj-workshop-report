---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Hoàn thiện tài liệu thiết kế kiến trúc hệ thống AWS
* Tổng kết lại toàn bộ dịch vụ AWS/bên thứ ba đã sử dụng và bài học rút ra
* Đóng gói báo cáo workshop cuối kỳ (Hugo site) và publish qua GitHub Pages

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Viết tài liệu kiến trúc: sơ đồ hệ thống, bảng dịch vụ sử dụng, luồng xác thực, các biện pháp bảo mật đã áp dụng, hạn chế & định hướng nâng cấp | `27/07/2026` | 2 ngày |
| **2** | - Viết bài blog kỹ thuật (so sánh EC2 và Lambda cho bài toán startup nhỏ) | `29/07/2026` | 1 ngày |
| **3** | - Chuẩn bị báo cáo workshop dạng Hugo site: cấu hình lại `baseURL`, sửa nội dung từng mục cho khớp dự án thật, deploy qua GitHub Pages (GitHub Actions) | `30/07/2026` | 2 ngày |
| **4** | - Viết phần tự đánh giá (Self-evaluation) và tổng kết dự án | `01/08/2026` | 1 ngày |

### Kết quả đạt được tuần 8:

* Hoàn thành tài liệu kiến trúc đầy đủ (kèm sơ đồ hệ thống), liệt kê rõ 8 dịch vụ đã dùng: Amazon EC2, Amazon RDS, Amazon Cognito, Google Cloud OAuth 2.0, AWS IAM/CloudShell, GitHub, DuckDNS, Google Authenticator.
* Xuất bản thành công báo cáo workshop dưới dạng site tĩnh (Hugo) qua GitHub Pages, khắc phục lỗi `baseURL` sai và lỗi cú pháp `config.toml` phát sinh trong lúc build.
* Nhìn lại toàn bộ 8 tuần: xác nhận lựa chọn kiến trúc EC2 + RDS là phù hợp cho quy mô startup nhỏ hiện tại (so với việc chuyển sang Lambda/serverless, vốn phát sinh chi phí sàn cao hơn ở mức traffic thấp); điểm cần cải thiện tiếp không nằm ở việc chọn sai dịch vụ, mà ở quy trình vận hành: cần CI/CD, cần nâng cấp RDS lên chuẩn production, cần quản lý secrets chặt chẽ hơn.

### Hạn chế:

* Vì rút gọn lịch trình để vừa đủ 8 tuần, một số hạng mục nâng cao (CI/CD tự động, RDS Multi-AZ, Amazon S3 cho file tĩnh, rate-limit cho TOTP) chưa kịp triển khai — được liệt kê rõ trong tài liệu kiến trúc như định hướng phát triển tiếp theo sau kỳ thực tập.