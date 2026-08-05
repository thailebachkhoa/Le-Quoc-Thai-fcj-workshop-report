---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Thiết kế các UML diagram, luồng logic theo phạm vi đề ra
* Phác thảo database schema gắn với luồng xử lý
* Hoàn thành các phần thô (static) của web chạy local

### Các công việc cần triển khai trong tuần này:

| Thứ tự | Công việc | Ngày bắt đầu | Thời gian dự kiến |
| :-: | :--- | :-: | :-: |
| **1** | - Vẽ Usecase Diagram, Sequence Diagram, Activity Diagram (ưu tiên chi tiết nghiệp vụ kinh doanh non-tech trước) <br> - Diagram không nhất thiết phải vẽ hình — mô tả chi tiết bằng text cũng được, thuận tiện cho việc prompt các AI Agent hỗ trợ code | `15/06/2026` | 2 ngày |
| **2** | - Vẽ các diagram kiến trúc (class diagram, component diagram) <br> - Deployment Diagram trên AWS sẽ vẽ sau khi thử thành công web thô trên EC2 | `17/06/2026` | 2-3 ngày |
| **3** | - Viết schema.sql cho Database (ưu tiên các usecase/tính năng lõi: bán hàng) | `20/06/2026` | 2 ngày |

### Kết quả đạt được tuần 2:

* Hoàn thành "bản kiến trúc" trước khi bước vào xây dựng codebase, tạo tiền đề cho tài liệu thiết kế kiến trúc hoàn chỉnh ở tuần cuối (Tuần 8).
* Chốt được schema.sql cho các bảng lõi: `users`, `products`, `orders`, `news`, `comments`. 

### Hạn chế:

* Chưa kịp tiến độ chạy web thô trên localhost như kế hoạch — dời sang đầu Tuần 3.