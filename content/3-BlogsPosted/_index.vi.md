
---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây sẽ là phần liệt kê, giới thiệu các blogs đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). 

###  [Blog 1 - Backup database lên S3 mà không cần hardcode Access Key — nhờ IAM Role](3.1-Blog1/)
Blog này hướng dẫn cách sử dụng IAM Role gắn trực tiếp vào EC2 instance để tự động backup MySQL lên S3 mà không cần lưu Access Key hay Secret Key trong mã nguồn. Giải pháp này giúp áp dụng nguyên tắc Least Privilege hiệu quả và loại bỏ hoàn toàn rủi ro lộ credential bảo mật.

###  [Blog 2 - 3 cái bẫy chi phí AWS mà tân binh dễ dính (và cách né bằng Free Tier + tự động hoá)](3.2-Blog2/)
Blog này chia sẻ 3 tình huống thực tế dễ làm phát sinh chi phí ngoài dự kiến khi triển khai dự án cá nhân trên AWS: thay đổi chính sách phí IPv4/Elastic IP, tạo nhầm Region cho S3 bucket, và để EC2/RDS chạy liên tục 24/7[cite: 2]. Bài viết cũng hướng dẫn cách phòng tránh bằng Free Tier, tự động hóa tắt/mở tài nguyên với Lambda + EventBridge Scheduler, và thiết lập lưới an toàn bằng AWS Budgets.


###  [Blog 3 - Amazon Cognito, giải thích cho đàng hoàng: User Pool, Hosted UI, Federation, và trong token thật sự có gì](3.3-Blog3/)
Bài giải thích từ gốc về Amazon Cognito: khác biệt giữa User Pools và Identity Pools, các thành phần bên trong 1 User Pool (App Client, Domain, Identity Providers, Groups), luồng ai-nói-chuyện-với-ai từng bước khi đăng nhập liên kết qua Google, trong JWT phát hành ra thật sự chứa claim gì, và verify token đúng cách cần làm gì ngoài việc chỉ decode nó.
 