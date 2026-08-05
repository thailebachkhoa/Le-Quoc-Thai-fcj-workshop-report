---
title: "3. Mở rộng dịch vụ: S3, IAM, CloudWatch"
weight: 3
date: 2026-08-05
draft: false
---

## 3.1. Tạo IAM Role cho EC2 (Least Privilege)

**IAM → Policies → Create policy → JSON:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::<tên-bucket>",
                "arn:aws:s3:::<tên-bucket>/*"
            ]
        }
    ]
}
```

**IAM → Roles → Create role** → Trusted entity: AWS service → EC2 → gắn Policy trên → đặt tên `PlantifyEC2Role`.

Gắn vào EC2: **EC2 → Actions → Security → Modify IAM role** → chọn Role vừa tạo.

## 3.2. Tạo S3 bucket

**S3 → Create bucket** — chú ý chọn đúng **Region trùng với EC2/RDS** (`ap-southeast-1`), giữ nguyên **Block Public Access: ON**.

## 3.3. Script backup tự động

```bash
sudo tee /home/ubuntu/backup-db.sh > /dev/null << 'EOF'
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="/tmp/plantify_backup_$DATE.sql"

mysqldump -h <rds-endpoint> -u admin -p'<password>' \
  --single-transaction --set-gtid-purged=OFF plantify > $BACKUP_FILE

aws s3 cp $BACKUP_FILE s3://<tên-bucket>/
rm $BACKUP_FILE
EOF
sudo chmod +x /home/ubuntu/backup-db.sh
```

Kiểm tra Role hoạt động:
```bash
sudo apt install -y awscli
aws sts get-caller-identity
```

Tự động hóa bằng cron (chạy 2h sáng mỗi ngày):
```bash
sudo crontab -e
```
```
0 2 * * * /home/ubuntu/backup-db.sh >> /var/log/plantify-backup.log 2>&1
```

## 3.4. CloudWatch Alarm + SNS

**SNS → Topics → Create topic** (Standard) → **Create subscription** (Email) → xác nhận qua email.

**CloudWatch → Alarms → Create alarm:**

| Alarm | Metric | Điều kiện |
|---|---|---|
| `Plantify-EC2-HighCPU` | EC2 → CPUUtilization | Greater than 80 |
| `Plantify-RDS-LowStorage` | RDS → FreeStorageSpace | **Lower than** 2000000000 |

Cả 2 chọn Notification → topic SNS vừa tạo.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| Bucket tạo nhầm region US East thay vì Singapore | Không để ý dropdown Region mặc định | Xóa bucket, đợi vài phút, tạo lại đúng region — S3 không cho đổi region sau khi tạo |
| Script backup báo "Permission denied" dù đã `chmod +x` | File tạo qua `nano` dính ký tự ẩn/dòng shebang lỗi | Xóa và tạo lại bằng `cat heredoc` thay vì `nano` |
| `mysqldump` cảnh báo "inconsistent backup" | Thiếu cờ `--single-transaction` | Thêm `--single-transaction --set-gtid-purged=OFF` |
| CloudWatch Alarm báo động ngay dù hệ thống bình thường | Set nhầm điều kiện "Greater than" thay vì "Lower than" cho metric mà giá trị cao là tốt (FreeStorageSpace) | Đọc kỹ nội dung email "Reason for State Change", sửa lại đúng chiều so sánh |