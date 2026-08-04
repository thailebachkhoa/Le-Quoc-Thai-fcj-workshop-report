---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây sẽ là phần liệt kê, giới thiệu các blogs đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). 

###  [Blog 1 - Backup database lên S3 mà không cần hardcode Access Key — nhờ IAM Role](3.1-Blog1/)
Blog này hướng dẫn cách sử dụng IAM Role gắn trực tiếp vào EC2 instance để tự động backup MySQL lên S3 mà không cần lưu Access Key hay Secret Key trong mã nguồn[cite: 1]. Giải pháp này giúp áp dụng nguyên tắc Least Privilege hiệu quả và loại bỏ hoàn toàn rủi ro lộ credential bảo mật[cite: 1].

###  [Blog 2 - 3 cái bẫy chi phí AWS mà tân binh dễ dính (và cách né bằng Free Tier + tự động hoá)](3.2-Blog2/)
Blog này chia sẻ 3 tình huống thực tế dễ làm phát sinh chi phí ngoài dự kiến khi triển khai dự án cá nhân trên AWS: thay đổi chính sách phí IPv4/Elastic IP, tạo nhầm Region cho S3 bucket, và để EC2/RDS chạy liên tục 24/7[cite: 2]. Bài viết cũng hướng dẫn cách phòng tránh bằng Free Tier, tự động hóa tắt/mở tài nguyên với Lambda + EventBridge Scheduler, và thiết lập lưới an toàn bằng AWS Budgets[cite: 2].

###  [Blog 3 - CloudWatch Alarm tưởng dễ mà không dễ: câu chuyện set ngược điều kiện Greater/Lower Than](3.3-Blog3/)
Blog này phân tích một lỗi cấu hình nhỏ nhưng rất dễ mắc phải khi tạo CloudWatch Alarm giám sát dung lượng RDS (`FreeStorageSpace`) do chọn nhầm điều kiện "Greater Than" thay vì "Lower Than"[cite: 3]. Bài viết chia sẻ kinh nghiệm đọc log cảnh báo từ email, cách khắc phục và bài học kiểm thử Alarm hai chiều trước khi đưa vào vận hành[cite: 3].