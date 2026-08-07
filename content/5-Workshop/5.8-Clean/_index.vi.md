---
title: "8. Clean-up"
weight: 8
date: 2026-08-07
draft: false
---

## Clean-up

Thực hiện theo đúng thứ tự để tránh lỗi dependency:

1. **EventBridge Scheduler** — xóa 4 lịch trình trước (tránh Lambda tiếp tục gọi sau khi xóa resource)
2. **Lambda** — xóa function `plantify-scheduler`
3. **CloudWatch** — xóa 2 Alarm (`Plantify-EC2-HighCPU`, `Plantify-RDS-LowStorage`)
4. **SNS** — xóa subscription → xóa topic `plantify-alerts`
5. **Cognito** — xóa App Client → xóa User Pool
6. **EC2** — Stop instance → **Terminate** (không phải chỉ Stop)
7. **Elastic IP** — **Release** ngay sau khi Terminate EC2 (nếu không, tiếp tục bị tính phí)
8. **RDS** — xóa DB instance (bỏ tick "Create final snapshot" nếu không cần)
9. **S3** — **Empty bucket** trước → Delete bucket (không xóa được bucket còn object)
10. **IAM** — xóa Role `PlantifyEC2Role`, `PlantifySchedulerRole` → xóa Policy tương ứng
11. **Security Groups** — xóa các Security Group tự tạo (default SG không xóa được)
12. **VPC** — nếu đã tạo Default VPC riêng, có thể giữ lại (không tính phí khi rỗng)

> ⚠️ Kiểm tra lại **Billing → Cost Explorer** sau 24 giờ để xác nhận không còn phát sinh phí.