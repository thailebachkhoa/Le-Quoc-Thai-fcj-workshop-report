---
title: "Week 1 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Research documentation related to cloud computing relevant to the program
* Have an initial UI sketch/draft
* Select the technology and architecture for the project
* Build a study plan / roadmap spanning the full 8-week internship

### Tasks to be carried out this week:
| No. | Task | Estimated Time |
| --- | --- | --- |
| 1 | - Read and update on internal rules and regulations at the host unit <br> - Research documentation on Cloud Computing (AWS) | Daily |
| 2 | - Proactively consult senior students/social media to build an AWS learning roadmap | 7 day |
| 3 | - Create an AWS account and claim the $200 free credit <br> - Choose a topic and reference similar projects | 1 day |
| 4 | - Learn EC2 basics: <br>&emsp; + Instance types <br>&emsp; + AMI <br>&emsp; + EBS <br>&emsp; + ... <br> - Methods for remote SSH into EC2 <br> - Learn about Elastic IP | 1 week |
| 5 | - Develop ideas for the interface <br> - Review the Ubuntu CLI environment (the OS to be chosen on EC2) <br> - Demo a static website using Elastic IP | 1 week |

### Week 1 Achievements:

* **Research & classification of AWS services:**
  * **Core Services:** `EC2`, `Lambda` (Compute); `S3`, `EBS` (Storage); `VPC`, `Route 53` (Networking); `RDS`, `DynamoDB` (Database); `IAM` (Security).
  * **Supporting Services:** `ELB`, `Auto Scaling`, `CloudFront`, `ElastiCache`, `CloudWatch`, `CloudTrail`, `SQS`, `SNS`.
  * **Specialized Services:** `SageMaker`, `Redshift`, `ECS`, `EKS`, `CodePipeline`, ...
* **Environment initialization & configuration:**
  * Successfully registered an **AWS Free Tier** account with $200 in free credit.
  * Became familiar with and used the **AWS Management Console** to access and manage services via the web interface.
* **Built an onboarding roadmap**: EC2 -> RDS prioritized first — sufficient to deploy a basic website; advanced services (Cognito, Lambda...) to be learned in later weeks.
* **Built the UI structure based on sample websites**, reference sources:
  * thegioihoa.net
  * https://caycanhdian.com/
  * https://rlc.vn/
  * https://www.cayxanhdep.vn/
* **Tech stack selection**: Pure PHP (no framework), MySQL, standard MVC architecture — prioritizing rapid development given the short internship timeline.

### Limitations:

* Haven't yet learned AWS CLI and Lambda, so the project's core architecture is tightly fixed to EC2 -> will face difficulties with scalability if traffic spikes suddenly (analyzed further in the final self-evaluation, Week 8).