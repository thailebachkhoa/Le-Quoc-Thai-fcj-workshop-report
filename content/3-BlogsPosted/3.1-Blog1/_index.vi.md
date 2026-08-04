---
title: "Backup database lên S3 mà không cần hardcode Access Key — nhờ IAM Role"
date: 2026-08-04
draft: false
tags: ["aws", "iam", "s3", "security"]
description: "Cách dùng IAM Role gắn vào EC2 để tự động backup MySQL lên S3, không cần lưu Access Key/Secret Key ở bất kỳ đâu trong code."
---

## Vấn đề ban đầu

Khi làm dự án Plantify Co (web bán cây cảnh, PHP + MySQL trên AWS), mình cần một cơ chế backup database tự động lên S3. Cách "quen tay" nhất mà nhiều người mới học AWS hay làm là:

```php
$s3 = new S3Client([
    'credentials' => [
        'key'    => 'AKIAxxxxxxxxxxxxxxxx',
        'secret' => 'wJalrxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    ],
]);
```

Nhìn qua thì chạy được ngay, nhưng đây là một trong những sai lầm bảo mật phổ biến nhất: nếu code này lỡ đẩy lên GitHub public (dù chỉ 1 lần rồi xoá ngay), Access Key vẫn tồn tại vĩnh viễn trong lịch sử Git, và bot quét GitHub tìm access key lộ ra chỉ mất vài phút để phát hiện.

## Giải pháp: IAM Role thay vì Access Key

AWS có cơ chế hay hơn nhiều: gắn thẳng một **IAM Role** vào EC2 instance. Khi đó, mọi chương trình chạy trên instance đó (bao gồm AWS CLI, SDK...) tự động có được quyền tương ứng — không cần khai báo access key ở bất kỳ đâu trong code hay file cấu hình.

### Bước 1 — Viết Policy theo đúng nguyên tắc Least Privilege

Thay vì cấp quyền `s3:*` (toàn quyền trên mọi bucket), chỉ cấp đúng 3 action cần thiết, trên đúng 1 bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::plantify-backup-bucket",
                "arn:aws:s3:::plantify-backup-bucket/*"
            ]
        }
    ]
}
```

Nếu policy này lỡ bị lộ, kẻ tấn công cũng chỉ có thể đọc/ghi đúng 1 bucket backup — không đụng được vào bất kỳ tài nguyên AWS nào khác trong tài khoản.

### Bước 2 — Tạo Role, gắn Policy, gắn vào EC2

Tạo IAM Role với Trusted entity là **AWS service → EC2**, gắn Policy trên vào, rồi vào EC2 Console: **Actions → Security → Modify IAM role** → chọn Role vừa tạo. Không cần restart instance, Role có hiệu lực ngay.

### Bước 3 — Script backup không cần biết access key là gì

```bash
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="/tmp/plantify_backup_$DATE.sql"

mysqldump -h <rds-endpoint> -u admin -p'<password>' \
  --single-transaction --set-gtid-purged=OFF plantify > $BACKUP_FILE

aws s3 cp $BACKUP_FILE s3://plantify-backup-bucket/

rm $BACKUP_FILE
```

Lệnh `aws s3 cp` chạy được ngay, không có dòng nào khai báo credentials — AWS CLI tự động lấy credential tạm thời từ IAM Role đang gắn với instance.

## Kiểm chứng Role hoạt động

```bash
aws sts get-caller-identity
```

Nếu thấy trả về dạng `arn:aws:sts::...:assumed-role/PlantifyEC2Role/...`, nghĩa là instance đang "mượn" quyền từ Role, không dùng access key tĩnh nào cả.

## Bài học rút ra

- Không bao giờ hardcode Access Key/Secret Key trong code, kể cả trong file `.env` nếu file đó có khả năng lộ ra ngoài.
- IAM Role + Least Privilege là cách làm chuẩn khi ứng dụng chạy trên EC2/Lambda cần gọi tới dịch vụ AWS khác.
- Luôn giới hạn Policy ở mức tối thiểu cần thiết — chỉ đúng action, đúng resource, không rộng hơn.

Đặt lịch backup này chạy qua `cron` mỗi ngày, kết hợp CloudWatch Alarm giám sát dung lượng RDS, là một lớp bảo vệ dữ liệu tối thiểu nhưng hiệu quả cho bất kỳ dự án nhỏ nào mới lên cloud.