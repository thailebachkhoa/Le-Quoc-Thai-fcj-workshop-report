---
title: "Worklog Tuần 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

* Tìm kiếm tài liệu cloud computing liên quan chương trình
* Có bản vẽ phác thảo đầu về UI
* Lựa chọn công nghệ, kiến trúc cho dự án
* Xây dựng kế hoạch học tập - lộ trình xuyên suốt 8 tuần thực tập

### Các công việc cần triển khai trong tuần này:
| Thứ tự | Công việc | Thời gian dự kiến |
| --- | --- | --- |
| 1 | - Đọc và cập nhập các nội quy, quy định tại đơn vị thực tập <br> - Tìm kiếm tài liệu Cloud Computing về AWS | Mỗi ngày |
| 2 | - Chủ động tham khảo bạn bè khóa trước, MXH xây dựng lộ trình học tập AWS | 7 ngày |
| 3 | - Tạo tài khoản AWS và lấy 200$ free <br> - Chọn đề tài và tham khảo dự án tương tự | 1 ngày |
| 4 | - Tìm hiểu EC2 cơ bản: <br>&emsp; + Instance types <br>&emsp; + AMI <br>&emsp; + EBS <br>&emsp; + ... <br> - Các cách remote SSH vào EC2 <br> - Tìm hiểu Elastic IP | 1 tuần |
| 5 | - Phát triển ý tưởng cho giao diện <br> - Ôn lại môi trường CLI của Ubuntu (OS sẽ chọn trên EC2) <br> - Demo web tĩnh nhờ Elastic IP | 1 tuần |

### Kết quả đạt được tuần 1:

* **Nghiên cứu & phân loại các dịch vụ AWS:**
  * **Core Services (Cốt lõi):** `EC2`, `Lambda` (Compute); `S3`, `EBS` (Storage); `VPC`, `Route 53` (Networking); `RDS`, `DynamoDB` (Database); `IAM` (Security).
  * **Supporting Services (Bổ trợ):** `ELB`, `Auto Scaling`, `CloudFront`, `ElastiCache`, `CloudWatch`, `CloudTrail`, `SQS`, `SNS`.
  * **Specialized Services (Nâng cao):** `SageMaker`, `Redshift`, `ECS`, `EKS`, `CodePipeline`, ...
* **Khởi tạo & cấu hình môi trường:**
  * Đăng ký thành công tài khoản **AWS Free Tier** với 200$ miễn phí.
  * Làm quen và sử dụng **AWS Management Console** để truy cập, quản lý các dịch vụ trên giao diện web.
* **Xây dựng được roadmap cho nhập môn**: ưu tiên EC2 -> RDS trước tiên — đủ để deploy một web cơ bản, các dịch vụ nâng cao (Cognito, Lambda...) sẽ học ở các tuần sau.
* **Xây dựng được cấu trúc UI từ các web mẫu**, nguồn tham khảo:
  * thegioihoa.net
  * https://caycanhdian.com/
  * https://rlc.vn/
  * https://www.cayxanhdep.vn/
* **Lựa chọn công nghệ**: PHP thuần (không framework), MySQL, kiến trúc MVC phổ thông — ưu tiên phát triển nhanh trong thời gian thực tập ngắn.

### Hạn chế:

* Chưa học Lambda, nên kiến trúc lõi của dự án đang cố định vào EC2 -> sẽ gặp khó khăn về khả năng mở rộng nếu traffic tăng đột biến (đã phân tích rõ hơn ở phần đánh giá cuối dự án, Tuần 8).