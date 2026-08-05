---
title: "3. Expanding Services: S3, IAM, CloudWatch"
weight: 3
date: 2026-08-05
draft: false
---

## 3.1. Creating an IAM Role for EC2 (Least Privilege)

**IAM → Policies → Create policy → JSON:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::<bucket-name>",
                "arn:aws:s3:::<bucket-name>/*"
            ]
        }
    ]
}
```

**IAM → Roles → Create role** → Trusted entity type: AWS service → Use case: EC2 → Attach the policy created above → Name it `PlantifyEC2Role`.

Attach to EC2: **EC2 → Actions → Security → Modify IAM role** → Select the newly created Role.

## 3.2. Creating an S3 Bucket

**S3 → Create bucket** — Make sure to select the **same Region as EC2/RDS** (`ap-southeast-1`), keep **Block Public Access: ON** enabled.

## 3.3. Automated Backup Script

```bash
sudo tee /home/ubuntu/backup-db.sh > /dev/null << 'EOF'
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="/tmp/plantify_backup_$DATE.sql"

mysqldump -h <rds-endpoint> -u admin -p'<password>' \
  --single-transaction --set-gtid-purged=OFF plantify > $BACKUP_FILE

aws s3 cp $BACKUP_FILE s3://<bucket-name>/
rm $BACKUP_FILE
EOF
sudo chmod +x /home/ubuntu/backup-db.sh
```

Verify the Role is working:
```bash
sudo apt install -y awscli
aws sts get-caller-identity
```

Automate with cron (runs at 2 AM daily):
```bash
sudo crontab -e
```
```
0 2 * * * /home/ubuntu/backup-db.sh >> /var/log/plantify-backup.log 2>&1
```

## 3.4. CloudWatch Alarm + SNS

**SNS → Topics → Create topic** (Standard) → **Create subscription** (Email) → Confirm via email.

**CloudWatch → Alarms → Create alarm:**

| Alarm | Metric | Condition |
|---|---|---|
| `Plantify-EC2-HighCPU` | EC2 → CPUUtilization | Greater than 80 |
| `Plantify-RDS-LowStorage` | RDS → FreeStorageSpace | **Lower than** 2000000000 |

For both alarms, select Notification → choose the newly created SNS topic.

## Common Issues

| Issue | Cause | Solution |
|---|---|---|
| Bucket accidentally created in US East instead of Singapore | Overlooked the default Region dropdown | Delete the bucket, wait a few minutes, recreate it in the correct region — S3 does not allow region changes after creation |
| Backup script returns "Permission denied" despite running `chmod +x` | File created via `nano` contains hidden characters or an invalid shebang line | Delete and recreate using `cat heredoc` instead of `nano` |
| `mysqldump` warns of "inconsistent backup" | Missing `--single-transaction` flag | Add `--single-transaction --set-gtid-purged=OFF` |
| CloudWatch Alarm triggers immediately even though the system is normal | Set the condition to "Greater than" instead of "Lower than" for a metric where higher values are good (FreeStorageSpace) | Read the "Reason for State Change" in the email notification carefully and reverse the comparison logic |