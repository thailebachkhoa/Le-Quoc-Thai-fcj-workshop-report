---
title: "Published Blog Posts"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section lists and introduces the blog posts published on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj).

###  [Blog 1 - Backup database to S3 without hardcoding Access Keys — using IAM Roles](3.1-Blog1/)
This blog demonstrates how to attach an IAM Role directly to an EC2 instance to automatically backup MySQL to S3 without storing Access Keys or Secret Keys anywhere in the code. This solution effectively enforces the Principle of Least Privilege and completely eliminates the risk of security credential leaks.

###  [Blog 2 - 3 AWS Cost Traps Beginners Easily Fall Into (And How to Avoid Them with Free Tier + Automation)](3.2-Blog2/)
This blog shares 3 real-world scenarios that can easily lead to unexpected costs when deploying personal projects on AWS: public IPv4/Elastic IP policy changes, creating S3 buckets in the wrong Region, and leaving EC2/RDS running 24/7. It also guides you on how to avoid these issues using the Free Tier, automating start/stop schedules with Lambda + EventBridge Scheduler, and setting up safety nets with AWS Budgets.

###  [Blog 3 - CloudWatch Alarms Aren't as Easy as They Seem: The Story of Setting the Greater/Lower Than Condition Backward](3.3-Blog3/)
This blog analyzes a small but common configuration mistake when creating a CloudWatch Alarm to monitor RDS storage (`FreeStorageSpace`) by mistakenly selecting the "Greater Than" condition instead of "Lower Than". It shares key insights on reading alert logs from email, how to fix the issue, and lessons learned on testing Alarms bidirectionally before going into production.