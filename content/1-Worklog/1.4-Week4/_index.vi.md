---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Sửa lỗi và hoàn chỉnh codebase
* Hoàn thành chạy thành công toàn bộ chức năng trên localhost
* Tìm hiểu thêm giải pháp tiết kiệm chi phí AWS cho giai đoạn deploy sắp tới

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| --- | --- | :-: | --- |
| 1 | - Tiếp tục hoàn thiện Backend (giỏ hàng, đơn hàng, quản trị sản phẩm) | `29/06/2026` | 4 ngày |
| 2 | - Tiếp tục mở rộng các trang web tĩnh, tính năng phụ (bình luận, đánh giá) | `29/06/2026` | Mỗi ngày |
| 3 | - Học cách tích hợp Lambda + EventBridge nhằm tắt EC2 theo lịch (tiết kiệm chi phí ngoài giờ demo) | `03/07/2026` | 1 ngày |

### Kết quả đạt được tuần 4:

* Hoàn thiện sản phẩm chạy ổn định trên môi trường localhost (XAMPP), đầy đủ luồng: đăng ký/đăng nhập cơ bản, xem sản phẩm, giỏ hàng, đặt hàng, khu vực admin quản lý sản phẩm/tin tức.
* Nắm được cách dùng Lambda + EventBridge để lên lịch tắt/bật EC2 tự động — chuẩn bị áp dụng thực tế khi có EC2 thật ở Tuần 5.

### Hạn chế:

* Codebase vẫn đang là PHP thuần, chưa viết test tự động — việc kiểm thử ở các tuần sau chủ yếu làm thủ công.