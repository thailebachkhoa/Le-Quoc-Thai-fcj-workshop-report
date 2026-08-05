---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Provision real AWS infrastructure (EC2, RDS) to replace localhost
* Ship the first live deployment of the website, with a domain and HTTPS
* Set up a deployment workflow from GitHub to EC2

### Tasks to be carried out this week:


| No. | Task | Start Date | Estimated Time |
| :-: | :--- | :-: | :-: |
| **1** | - Provision an EC2 instance (Ubuntu, t4g.micro), configure Security Group (open ports 80/443/22) <br> - Install Apache + PHP 8 on EC2 <br> - Register a free domain via DuckDNS, point it to the EC2 Elastic IP <br> - Configure HTTPS (Let's Encrypt/certbot) | `06/07/2026` | 2 days |
| **2** | - Provision Amazon RDS (MySQL 8.4, t4g.micro), configure Security Group to allow EC2 to connect to RDS <br> - Import schema.sql into RDS <br> - Deploy code from GitHub to EC2 (git clone/git pull), configure `.env`, test the full flow that already worked on localhost | `08/07/2026` | 1 day |
| **3** | - Write a Lambda function based on context and event parameters <br> - Create a separate IAM role for the above Lambda function (with permissions to only stop/start specific resources) | `09/07/2026` | 2 days |
| **4** | - Create EventBridge Scheduler with 4 fixed daily schedules, each pointing to the same Lambda but with different JSON payloads <br> - Add a try/except block to avoid invoking overlapping RDS states <br> - Mechanism: Scheduler -(calls)-> Lambda function --(AWS SDK for Python)-> Stop/Start EC2/RDS (modify the error-index file in EC2 to notify that the website is closed, ensuring UX) | `10/07/2026` | 1 day |


### Week 5 Achievements:

* The website is live on the real internet at `i-love-fcaj.duckdns.org`, with valid HTTPS.
* Compute (EC2) and database (RDS) are separated instead of running on a single machine — easier to back up and scale each part independently later on.
* Established a manual deployment workflow: edit code -> push to GitHub -> SSH into EC2 -> `git pull` -> restart Apache.
* The Database and  the server stops daily at 0h00 am and re-open at 7h am 

### Limitations:

* Deployment is entirely manual, with no CI/CD yet — every code update requires repeating all the steps by hand, and it's easy to forget a step (e.g., forgetting to reset the `www-data` file ownership after pulling), which caused a few minor issues that had to be resolved in Week 7.
* RDS is currently configured as Single-AZ with only a 1-day backup retention — sufficient for the demo stage, but not yet up to production standards.