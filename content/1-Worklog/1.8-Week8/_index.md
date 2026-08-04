---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:

* Finalize the system architecture design documentation
* Summarize all the AWS/third-party services used and the lessons learned
* Package the final workshop report (Hugo site) and publish it via GitHub Pages

### Tasks to be carried out this week:

| No. | Task | Start Date | Estimated Time |
| :-: | :--- | :-: | :-: |
| **1** | - Write the architecture documentation: system diagram, table of services used, authentication flow, security measures applied, limitations & future improvements | `27/07/2026` | 2 days |
| **2** | - Write a technical blog post (comparing EC2 and Lambda for a small startup's use case) | `29/07/2026` | 1 day |
| **3** | - Prepare the workshop report as a Hugo site: reconfigure `baseURL`, edit each section's content to match the real project, deploy via GitHub Pages (GitHub Actions) | `30/07/2026` | 2 days |
| **4** | - Write the self-evaluation and final project summary | `01/08/2026` | 1 day |

### Week 8 Achievements:

* Completed the full architecture documentation (with a system diagram), clearly listing the 8 services used: Amazon EC2, Amazon RDS, Amazon Cognito, Google Cloud OAuth 2.0, AWS IAM/CloudShell, GitHub, DuckDNS, and Google Authenticator.
* Successfully published the workshop report as a static site (Hugo) via GitHub Pages, fixing an incorrect `baseURL` and a `config.toml` syntax error that came up during the build.
* Looking back over the full 8 weeks: confirmed that the EC2 + RDS architecture choice was appropriate for the current small-startup scale (compared to moving to Lambda/serverless, which incurs a higher cost floor at low traffic); the remaining room for improvement isn't about choosing the wrong service, but about the operational process: CI/CD is needed, RDS needs to be upgraded to production standards, and secrets need to be managed more carefully.

### Limitations:

* Because the schedule was condensed to fit exactly 8 weeks, some advanced items (automated CI/CD, RDS Multi-AZ, Amazon S3 for static files, TOTP rate-limiting) were not implemented in time — they are clearly listed in the architecture documentation as directions for future development beyond the internship.