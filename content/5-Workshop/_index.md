---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Overview

Plantify Co is a plant e-commerce web application built with vanilla PHP using a custom MVC architecture and MySQL as the database. This workshop documents the complete process of deploying an existing local application to a real AWS environment, following the exact sequence in which the deployment was performed—including mistakes, rollbacks, and design changes made along the way.

The objectives of this workshop are:

* Deploy a traditional PHP application to AWS using a **two-tier architecture (EC2 + RDS)** without serverless, making it suitable for an existing PHP codebase.
* Integrate **5+ AWS services**, with each service serving a clear architectural purpose rather than being included only for demonstration.
* Achieve three key goals: **security** (least privilege, HTTPS, CSRF protection), **observability** (CloudWatch, SNS), and **cost optimization** (automatic resource start/stop scheduling).
* Replace the custom authentication system with **Amazon Cognito + Google OAuth**, with mandatory **TOTP-based MFA for administrators**.

Source code: https://github.com/thailebachkhoa/FCAJ-Intern-Project

Demo website: hosted on a DuckDNS domain with HTTPS enabled through Let's Encrypt.

# Prerequisites

Before starting, prepare the following:

| Item                 | Notes                                                                                                                        |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| AWS account          | Preferably within the AWS Free Tier (first 12 months) to minimize costs                                                      |
| AWS Region           | `ap-southeast-1` (Singapore) — used consistently across all services to reduce latency and avoid cross-region transfer costs |
| SSH client           | OpenSSH (WSL, Git Bash, PowerShell, or Terminal)                                                                             |
| Git                  | For cloning the repository and synchronizing GitHub ↔ EC2                                                                    |
| Google Cloud account | To create the OAuth Client used by Cognito for Google sign-in                                                                |
| Free domain          | DuckDNS or a similar service, required for issuing a Let's Encrypt SSL certificate                                           |
| Basic knowledge      | HTML/CSS/PHP, MVC architecture, and a basic understanding of VPC and Security Groups                                         |

No prior knowledge of Terraform, AWS CDK, Docker, or Kubernetes is required. This workshop uses the AWS Console to make each deployment step easier to understand.

# Architecture

## AWS infrastructure


![Approve images](Y.jpg "Approve images")


## AWS services and responsibilities

| AWS Service                  | Purpose                                               |
| ---------------------------- | ----------------------------------------------------- |
| Amazon EC2                   | Runs Apache, PHP 8.5, and ffmpeg                      |
| Amazon RDS (MySQL)           | Database server in a private subnet                   |
| Amazon S3                    | Database backup storage                               |
| AWS IAM                      | Least-privilege roles and policies for EC2 and Lambda |
| Amazon CloudWatch            | Monitors EC2 CPU usage and RDS storage utilization    |
| Amazon SNS                   | Sends email alerts                                    |
| AWS Lambda                   | Implements automated start/stop logic                 |
| Amazon EventBridge Scheduler | Executes Lambda functions on a schedule               |
| Amazon Cognito               | User management, Hosted UI, and Admin/Member groups   |
| Google Cloud OAuth           | External identity provider for Google authentication  |

## Application architecture (MVC)

```text
Browser
   │
   ▼
index.php (router)
   │
   ▼
Controller (Authentication + CSRF validation)
   │
   ├──────────────┐
   ▼              ▼
Model (PDO)    View (HTML rendering)
   │              ▲
   └──────┬───────┘
          ▼
      Amazon RDS (MySQL)
```

# Workshop modules

The workshop is divided into **seven parts**, following the exact order in which the project was implemented. You can read them sequentially or jump directly to a specific topic.

1. **Reviewing and hardening the application before moving to the cloud**
2. **Building the core infrastructure: EC2 + RDS**
3. **Adding supporting services: S3, IAM, and CloudWatch**
4. **Configuring a domain, HTTPS, and Elastic IP**
5. **Automating cost optimization with Lambda and EventBridge**
6. **Replacing authentication with Cognito, Google OAuth, and TOTP MFA**
7. **Fixing mobile UI issues and cleaning up unused code**
8. **Clean up**
Each module includes a **Common issues** section that documents the actual errors encountered during deployment and how they were resolved, allowing readers to avoid repeating the same troubleshooting process.
