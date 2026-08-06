
---
title: "Backup database to S3 without hardcoding Access Keys — using IAM Roles"
date: 2026-08-04
draft: false
tags: ["aws", "iam", "s3", "security"]
description: "How to attach an IAM Role to an EC2 instance to automatically backup MySQL to S3, without storing Access Key/Secret Key anywhere in the code."
---

## The Initial Problem

When working on the Plantify Co project (a plant-selling website, PHP + MySQL on AWS), I needed an automated database backup mechanism to S3. The most "habitual" approach that many AWS beginners take is:

```php
$s3 = new S3Client([
    'credentials' => [
        'key'    => 'AKIAxxxxxxxxxxxxxxxx',
        'secret' => 'wJalrxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    ],
]);
```

At first glance, it works immediately, but this is one of the most common security mistakes: if this code is accidentally pushed to a public GitHub repository (even just once and then deleted immediately), the Access Key remains permanently in the Git history, and bots scanning GitHub for exposed access keys will take only a few minutes to detect it.

## The Solution: IAM Role instead of Access Key

AWS has a much better mechanism: attaching an **IAM Role** directly to the EC2 instance. Then, any program running on that instance (including AWS CLI, SDKs...) automatically gets the corresponding permissions — no need to declare access keys anywhere in the code or configuration files.

### Step 1 — Writing a Policy following the Least Privilege principle

Instead of granting `s3:*` permissions (full access to all buckets), grant only the 3 necessary actions, on exactly 1 bucket:

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

If this policy is accidentally exposed, an attacker could only read/write to exactly 1 backup bucket — without being able to touch any other AWS resources in the account.

### Step 2 — Create the Role, attach the Policy, and attach it to EC2

Create an IAM Role with the Trusted entity set to **AWS service → EC2**, attach the above Policy, then go to the EC2 Console: **Actions → Security → Modify IAM role** → select the newly created Role. No instance restart is required; the Role takes effect immediately.

### Step 3 — Backup script that doesn't need to know what an access key is

```bash
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="/tmp/plantify_backup_$DATE.sql"

mysqldump -h <rds-endpoint> -u admin -p'<password>'   --single-transaction --set-gtid-purged=OFF plantify > $BACKUP_FILE

aws s3 cp $BACKUP_FILE s3://plantify-backup-bucket/

rm $BACKUP_FILE
```

The `aws s3 cp` command runs right away without a single line declaring credentials — the AWS CLI automatically fetches temporary credentials from the IAM Role currently attached to the instance.

## Verifying the Role works

```bash
aws sts get-caller-identity
```

If the response returns a format like `arn:aws:sts::...:assumed-role/PlantifyEC2Role/...`, it means the instance is "borrowing" permissions from the Role and isn't using any static access keys.

## Key Takeaways

- Never hardcode Access Key/Secret Key in your code, even in a `.env` file if that file has any chance of being exposed.
- IAM Role + Least Privilege is the standard practice when an application running on EC2/Lambda needs to call other AWS services.
- Always limit the Policy to the absolute minimum necessary — exact actions, exact resources, nothing broader.

Scheduling this backup to run daily via `cron`, combined with a CloudWatch Alarm to monitor RDS storage, forms a minimal but highly effective data protection layer for any small project newly migrated to the cloud.
