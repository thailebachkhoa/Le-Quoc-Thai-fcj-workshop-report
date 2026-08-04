---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Nhanh chóng hoàn thành FE, tối thiểu là trang cửa hàng - sản phẩm
* Thực hiện BE dựa trên các UML đã thiết kế
* Thực hành deploy code mẫu theo tài liệu đã tìm hiểu

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Hoàn thiện web thô (đặc biệt là Views trong MVC) và demo trên XAMPP <br> - Gợi ý: dựa trên UI các website mẫu, kết hợp AI hỗ trợ để phát triển nhanh nhất | `22/06/2026` | 3 ngày |
| **1.1** | - Mở rộng các trang tĩnh như FAQ hay News | `22/06/2026` | Mỗi ngày |
| **2** | - Tận dụng template MVC của các assignment cũ, phát triển Controller và Core trong MVC (nhiệm vụ BE) | `25/06/2026` | 3 ngày |
| **3** | - Học thêm các dịch vụ phụ trợ: AWS IAM, Cognito (phục vụ tính năng đăng nhập bằng Google), tạo thêm Elastic IP và tìm hiểu xây dựng HTTPS qua dịch vụ thứ 3 DuckDNS | `28/06/2026` | 1 ngày |

### Kết quả đạt được tuần 3:

* Chạy được tính năng cốt lõi của dự án: trang sản phẩm - giỏ hàng trên localhost.
* Thành công tái sử dụng mã nguồn MVC để phát triển nhanh dự án.

### Hạn chế:

Đây là đánh đổi từ kiến trúc đã chọn:

* Khả năng mở rộng bị giới hạn (phù hợp với khách hàng là startup nhỏ — thuần bán hàng, thay vì nhiều tầng nghiệp vụ như các big-tech).
* Khả năng chịu tải hệ thống sẽ kém hơn (vì các nghiệp vụ mua bán vào giờ cao điểm, lẽ ra nên để Lambda xử lý, lại đang được kiến trúc MVC nhiều lớp trên EC2 gánh hết, dễ gây quá tải CPU/RAM) — vấn đề đã đề cập từ Tuần 1.