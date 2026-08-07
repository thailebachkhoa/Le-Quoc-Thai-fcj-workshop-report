---
title: "8. Clean-up"
weight: 8
date: 2026-08-07
draft: false
---

## Clean-up

Follow the exact order to avoid dependency errors:

1. **EventBridge Scheduler** — Delete the 4 schedules first (prevents Lambda from being invoked after resource deletion)
2. **Lambda** — Delete the `plantify-scheduler` function
3. **CloudWatch** — Delete the 2 Alarms (`Plantify-EC2-HighCPU`, `Plantify-RDS-LowStorage`)
4. **SNS** — Delete subscription → Delete the `plantify-alerts` topic
5. **Cognito** — Delete App Client → Delete User Pool
6. **EC2** — Stop instance → **Terminate** (do not just Stop)
7. **Elastic IP** — **Release** immediately after terminating EC2 (otherwise, you will continue to be charged)
8. **RDS** — Delete DB instance (uncheck "Create final snapshot" if not needed)
9. **S3** — **Empty bucket** first → Delete bucket (a bucket containing objects cannot be deleted)
10. **IAM** — Delete `PlantifyEC2Role`, `PlantifySchedulerRole` roles → Delete corresponding Policies
11. **Security Groups** — Delete custom Security Groups (default SGs cannot be deleted)
12. **VPC** — If you created a custom VPC, you can keep it (no charge when empty)

> ⚠️ Check **Billing → Cost Explorer** after 24 hours to confirm no further charges are incurred.