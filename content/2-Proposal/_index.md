---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Plantify Co — A Secure, Self-Hosted E-Commerce Platform on AWS
## Deploying a PHP Plant Shop with Federated Login, Automated Backups, and Monitoring

### 1. Executive Summary
Plantify Co is a plant e-commerce and community website for green-lovers, built on a custom PHP MVC application (PDO + MySQL) that already existed as a codebase. 

**The goal** of this internship project is not to build the application from scratch, but to take it from "code on a laptop" to a **properly operated cloud service**: hosted on EC2, backed by a managed RDS database, protected by IAM least-privilege roles, backed up automatically to S3, monitored with CloudWatch alarms, and secured with federated login (Amazon Cognito + Google OAuth) plus TOTP two-factor authentication for admin accounts. 

The result is a small but realistically production-shaped deployment — the kind of stack a small business would actually run — built and operated end-to-end by a single intern within an 8-week window.

### 2. Problem Statement
### What's the Problem?
The Plantify application originally ran only as local PHP code with no cloud infrastructure: no managed database, no automated backups, no monitoring, and a login system limited to a plain username/password form with no second factor — meaning a single leaked admin password could compromise the whole store. There was also no repeatable, documented way to deploy or recover the system if a server was lost.

### The Solution
Move the application onto a minimal but real AWS architecture: **EC2** hosts the PHP app behind a DuckDNS domain, **RDS (MySQL)** replaces the local database, **IAM roles** (not static access keys) grant the EC2 instance only the permissions it needs, **S3** stores automated daily database backups triggered by a cron job, and **CloudWatch + SNS** raise alarms on abnormal CPU usage, low RDS free storage, and connection spikes. On top of this, authentication is upgraded from a plain login form to **Amazon Cognito with Google as a federated identity provider**, and admin accounts are additionally protected by **TOTP-based two-factor authentication**, so a compromised Google password alone is not enough to reach the admin dashboard.

### Benefits and Return on Investment
This project gives a working reference architecture for deploying a real PHP/MySQL application on AWS the "right way" — least-privilege IAM, automated backups, monitoring with alerting, and layered authentication — instead of the common shortcut of a single unmonitored VPS with root SSH access and a static password. It reduces operational risk (no more "did the backup actually run?" uncertainty), reduces the blast radius of a leaked password via 2FA, and produces a documented, repeatable deployment that can be rebuilt if the instance is lost. Running costs are kept to AWS Free-Tier-eligible or near-zero services (t-class EC2, small RDS instance, a handful of CloudWatch alarms, S3 storage in the tens of KB per backup), making this sustainable to keep running after the internship ends.

### 3. Solution Architecture
The platform runs on a straightforward AWS stack: end users reach the PHP application on an EC2 instance via a DuckDNS domain; the application reads and writes to a MySQL database on RDS; a cron job on EC2 performs a `mysqldump` (with `--single-transaction` for a consistent snapshot) and uploads the result to S3 using an IAM role attached to the instance (no embedded access keys); CloudWatch collects EC2 and RDS metrics and, through an SNS topic, emails the team when CPU usage, RDS storage, or RDS connections cross defined thresholds. Authentication for both members and admins is handled by Amazon Cognito's Hosted UI, federating to Google as the identity provider; admin accounts pass through an additional TOTP verification step before a privileged session is granted. Otherwise, **AWS Lambda** and **EventBride** will be used to fullfill the blank of EC2 ( run in low time )

### AWS Services Used
- **Amazon EC2**: Hosts the PHP MVC application (Plantify Co) and the backup cron job.
- **Amazon RDS (MySQL)**: Managed relational database for products, orders, users, comments, news, and FAQ content.
- **Amazon S3**: Stores automated, timestamped daily database backups.
- **AWS IAM**: Provides a least-privilege role attached to EC2 (no static access keys) for S3 access.
- **Amazon CloudWatch + Amazon SNS**: Monitors EC2 CPU, RDS free storage, and RDS connection count; sends email alarms.
- **Amazon Cognito**: Hosted authentication and session/token handling, federated with Google OAuth.
- **Google Cloud OAuth Client**: Identity provider for "Sign in with Google."
- **EventBride + Lambda**: Put EC2 and RDS in schedule

### Component Design
- **Web Application (EC2)**: Custom PHP MVC app — Controllers, Core (Auth middleware, PDO Database singleton, Env loader, Helpers), Models, and Views — serving the storefront, member dashboard, and admin panel.
- **Database (RDS)**: Stores products, orders (created via SQL transactions after re-validating prices server-side), users, comments (with pending/approved/hidden moderation states), news, FAQ, and site content.
- **Backup Pipeline (EC2 → IAM → S3)**: A scheduled cron job (`0 2 * * *`) runs `mysqldump --single-transaction`, saves a timestamped `.sql` file, and uploads it to S3 through the instance's IAM role.
- **Monitoring (CloudWatch → SNS)**: Alarms on `CPUUtilization` (EC2), `FreeStorageSpace` and `DatabaseConnections` (RDS) notify the team by email via an SNS topic.
- **Authentication (Cognito ↔ Google ↔ EC2)**: Browser redirects through Cognito Hosted UI to Google for login; Cognito exchanges the authorization code for tokens server-to-server; the PHP app verifies the JWT, reads the user's group, and — for Admins only — requires a TOTP code (secret stored in RDS, independent of the Google account) before granting a privileged session.
- **Resource automation (EventBridge → Lambda → EC2/RDS)**: EventBridge Scheduler triggers the `plantify-scheduler` Lambda function on four fixed daily schedules (08:00 start, 17:30 stop, 20:00 start, 00:00 stop — UTC+7); Lambda uses `boto3` to call `start_instances`/`stop_instances` for EC2 and `start_db_instance`/`stop_db_instance` for RDS, wrapping `InvalidDBInstanceStateFault` in a `try/except` block so no error is thrown when a resource is already in the target state. A dedicated IAM Role (`PlantifySchedulerRole`) restricts Lambda to exactly one EC2 instance and one RDS DB instance — no access to any other AWS resource in the account.
![Approve images](Y.jpg "Approve images")

### 4. Technical Implementation
**Implementation Phases**
The project follows four phases across the internship:
- Review the existing Plantify PHP/MySQL codebase and design the target AWS architecture 
- Provision core infrastructure — EC2, RDS, IAM roles, DuckDNS domain — and get the application running in the cloud for the first time 
- Layer on operational and security features — S3 automated backups with cron, CloudWatch alarms with SNS notifications, and Cognito + Google + TOTP authentication 
- Debug, harden, and document — resolve real integration issues (OAuth scope errors, missing identity providers, SQL binding bugs), then write up the architecture and deployment guide 

**Technical Requirements**
- **Application layer**: PHP 8.x with `pdo_mysql`, `fileinfo`, and `mbstring` extensions; MySQL/MariaDB; Apache with `mod_rewrite`.
- **Infrastructure layer**: Practical use of EC2 (instance setup, security groups), RDS (endpoint configuration, storage sizing), S3 (bucket policy, IAM-role-based access instead of access keys), and CloudWatch/SNS (metric alarms, email subscriptions).
- **Identity layer**: Cognito User Pool with a Google federated identity provider (correct `redirect_uri` and `SupportedIdentityProviders` configuration), plus a self-issued TOTP secret per admin user stored in RDS and verified server-side.

### 5. Timeline & Milestones
**Project Timeline (8 weeks)**
- Week 1–2: Research cloud fundamentals and review the existing Plantify codebase.
- Week 3–4: Stand up EC2 and RDS; deploy the application for the first time; configure IAM and DuckDNS.
- Week 5: Build the S3 backup pipeline and automate it with cron.
- Week 6: Integrate Cognito, Google OAuth, and TOTP two-factor authentication for admins.
- Week 7: Debug real-world integration issues (OAuth `invalid_scope`, missing identity provider, SQL bind errors) and configure CloudWatch alarms via SNS.
- Week 8: Finalize documentation, architecture diagrams, and knowledge-sharing blog posts; deploy the Hugo report site.

### 6. Budget Estimation
All services used are eligible for AWS Free Tier or run at near-zero cost at this project's scale (a single small EC2 instance, one small RDS instance, S3 storage in the tens of KB per daily backup, and a handful of CloudWatch alarms with SNS email notifications). No paid third-party services are required; Google OAuth client credentials and DuckDNS are free.

### Infrastructure Costs
- **EC2** (t-class instance): Free Tier eligible / minimal on-demand cost.
- **RDS** (small MySQL instance): Free Tier eligible / minimal on-demand cost.
- **S3**: Negligible — each backup file is tens of KB, stored daily.
- **CloudWatch + SNS**: Free Tier covers the small number of alarms and email notifications used.
- **Cognito**: Free for the small number of monthly active users in this project.
- **Domain**: DuckDNS is free.

### 7. Risk Assessment
#### Risk Matrix
- Inconsistent database backups during active writes: Medium impact, medium probability (mitigated by `--single-transaction`).
- Leaked admin credentials (e.g., Google account password): High impact, low-to-medium probability.
- Missed or silent backup/monitoring failures: Medium impact, medium probability.
- OAuth/identity-provider misconfiguration during setup: Low impact, high probability (expected during initial integration).

#### Mitigation Strategies
- Backups: Use `mysqldump --single-transaction` for a consistent snapshot even while the app is live; automate via cron; log every run.
- Credential leaks: Require TOTP two-factor authentication for all Admin accounts, independent of the Google password.
- Silent failures: CloudWatch alarms with SNS email notifications for CPU, RDS storage, and RDS connections; log file for backup runs.
- Configuration errors: Document exact Cognito App Client and Google OAuth Client settings (redirect URI, supported identity providers, scopes) to make the setup repeatable.

#### Contingency Plans
- If RDS storage runs low, scale storage or prune old data before it affects write availability.
- If the EC2 instance is lost, redeploy from the documented setup steps and restore the latest S3 backup.
- If Cognito/Google integration breaks, fall back temporarily to the application's local authentication path while re-configuring.

### 8. Expected Outcomes
#### Technical Improvements
A previously local-only PHP application becomes a monitored, backed-up, IAM-secured cloud deployment with federated login and admin two-factor authentication — a realistic small-scale production setup rather than a bare VPS.

#### Long-term Value
A documented, repeatable AWS deployment pattern (EC2 + RDS + S3 backups + CloudWatch alarms + Cognito/Google/TOTP auth) that can be reused for future PHP or general web projects, plus a set of knowledge-sharing blog posts and an 8-week worklog documenting real issues encountered and resolved.