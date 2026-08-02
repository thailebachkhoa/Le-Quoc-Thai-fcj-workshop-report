---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Nhanh chóng hoàn thành FE, tối thiểu là trang cửa hàng - sản phẩm
* Thực hiện BE trên các uml 
* Thực hành deploy code mẫu theo hướng dẫn tài liệu đã tìm kiếm


### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Hoàn thiện web thô (đặc biệt là Views trong MVC) và demo trên XAMPP <br> - Gợi ý: dựa trên UI các web site mẫu kết hợp các AI tích hợp để phát triển nhanh nhất | `29/06/2026` | 3 ngày |
| **1.1** | - Mở rộng các trang tĩnh như FAQ hay News | `29/06/2026` | Mỗi ngày |
| **2** | - Tận dụng template MVC của các Assignment cũ, phát triển Controller và Core trong MVC ( Nhiệm vụ BE ) | `2/07/2026` | 3 ngày |
| **3** | - Học thêm các dịch vụ phụ trợ: AWS IAM, Cognito (phục vụ tính năng đăng nhập bằng Google), tạo thêm elastic IP và xây dựng HTTPS qua dịch vụ thứ 3 DuckDNS | `3/7/2026' | 1 ngày |

### Kết quả đạt được tuần 3:

* Chạy được tính năng cốt lõi của dự án: trang sản phẩm - giỏ hàng trên localhost
* Thành công tái sử dụng mã nguồn MVC phát triển nhanh dự án 

### Điểm trừ

* Đây là đánh đổi từ kiến trúc đã chọn: 

* khả năng mở rộng (phù hợp với khách hàng là startup nhỏ: thuần bán hàng thay vì nhiều nghiệp vụ logic như các big-tech) 

* chịu tải hệ thống sẽ kém đi ( vì các nghiệp vụ mua bán trong giờ cao điểm nhẽ ra nên được lambda phụ trách lại được kiến trúc MVC nhiều lớp trong EC2 xử lí gây quá tải CPU RAM ) - vấn đề đã đề cập ở tuần 1