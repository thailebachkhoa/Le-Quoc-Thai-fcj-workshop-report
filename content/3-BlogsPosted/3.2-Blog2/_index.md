
---
title: "3 AWS Cost Traps Beginners Easily Fall Into (And How to Avoid Them with Free Tier + Automation)"
date: 2026-08-04
draft: false
tags: ["aws", "cost-optimization", "free-tier"]
description: "Three real-world scenarios where unexpected AWS costs arise in personal projects, and how to prevent them from the start."
---

When working on your first AWS project, the biggest fear isn't "how to make it work," but "how to avoid unexpected charges." Here are 3 real situations encountered when deploying Plantify Co to AWS, along with how to handle them.

## 1. Elastic IP — Free, but Not Forever

Many older documents (and even AIs, including myself initially) often say "Elastic IP is free when attached to a running instance." This is **no longer completely true** as of February 1, 2024 — AWS changed its policy: **all public IPv4 addresses are now charged** around $0.005/hour (~$3.65/month), regardless of whether they are attached to a running instance or not.

The better news: EC2 Free Tier includes 750 hours of public IPv4 address usage per month for the first 12 months — enough to cover a full month of 24/7 running with exactly 1 IP address. That means:

- During the first 12 months, using exactly 1 Elastic IP: still free.
- After 12 months, or if you have a 2nd IP address: charges will apply.

**Takeaway**: Don't rely blindly on outdated information about AWS pricing — pricing policies change quite frequently, so always double-check the official pricing page before making architectural decisions based on "what is free."

## 2. Creating an S3 Bucket in the Wrong Region — No Immediate Extra Cost, but Impacts Speed and Is Hard to Explain

When creating an S3 bucket for database backups, I accidentally left the default setting as **US East (N. Virginia)** while EC2 and RDS were both in **Singapore**. Technically it wasn't wrong — S3 can still be accessed cross-region — but:

- Data had to travel from Singapore to Virginia for every backup, increasing latency and slightly increasing data transfer fees.
- When presenting the architecture in a report, it was hard to justify why S3 was in a different region from the rest, other than "made a mistake."

**Takeaway**: Always double-check the Region dropdown **before** clicking Create — S3 does not allow changing regions after creation; you have to delete and recreate it from scratch if it's wrong.

## 3. Running EC2 + RDS 24/7 When Unnecessary

For a demo/learning project, you don't need servers running day and night. Solution: use **AWS Lambda + EventBridge Scheduler** to automatically Stop resources outside of active hours and Start them back up when needed.

```python
def lambda_handler(event, context):
    action = event.get('action')
    ec2 = boto3.client('ec2', region_name=REGION)
    rds = boto3.client('rds', region_name=REGION)

    if action == 'start':
        ec2.start_instances(InstanceIds=[EC2_INSTANCE_ID])
        rds.start_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
    elif action == 'stop':
        ec2.stop_instances(InstanceIds=[EC2_INSTANCE_ID])
        rds.stop_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
