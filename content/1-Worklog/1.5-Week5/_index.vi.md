---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Khởi tạo hạ tầng AWS thật (EC2, RDS) thay cho localhost
* Đưa được bản deploy đầu tiên của website lên internet, có domain và HTTPS
* Thiết lập quy trình deploy code từ GitHub lên EC2

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Khởi tạo EC2 instance (Ubuntu, t4g.micro), cấu hình Security Group (mở port 80/443/22) <br> - Cài đặt Apache + PHP 8 trên EC2 | `06/07/2026` | 2 ngày |
| **2** | - Khởi tạo Amazon RDS (MySQL 8.4, t4g.micro), cấu hình Security Group cho phép EC2 kết nối tới RDS <br> - Import schema.sql lên RDS | `08/07/2026` | 1 ngày |
| **3** | - Đăng ký domain miễn phí qua DuckDNS, trỏ về Elastic IP của EC2 <br> - Cấu hình HTTPS (Let's Encrypt/certbot) | `09/07/2026` | 1 ngày |
| **4** | - Đưa code từ GitHub lên EC2 (git clone/git pull), cấu hình `.env`, kiểm thử toàn bộ luồng đã chạy được trên localhost | `10/07/2026` | 2 ngày |

### Kết quả đạt được tuần 5:

* Website chạy được trên internet thật, tại domain `i-love-fcaj.duckdns.org`, có HTTPS hợp lệ.
* Tách riêng compute (EC2) và database (RDS) thay vì gộp chung 1 máy — dễ backup, dễ scale riêng từng phần sau này.
* Xác lập được quy trình deploy thủ công: sửa code -> push GitHub -> SSH vào EC2 -> `git pull` -> restart Apache.

### Hạn chế:

* Deploy hoàn toàn thủ công, chưa có CI/CD — mỗi lần cập nhật code phải tự tay lặp lại đủ các bước, dễ quên bước (ví dụ quên đổi quyền sở hữu file `www-data` sau khi pull), dẫn đến một số lỗi vặt phải xử lý ở Tuần 7.
* RDS đang cấu hình Single-AZ, backup retention chỉ 1 ngày — đủ cho giai đoạn demo nhưng chưa đạt chuẩn production thật.