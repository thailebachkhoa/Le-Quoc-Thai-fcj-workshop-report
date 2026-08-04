---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

* Deploy the full Google + Cognito + TOTP login flow to the production EC2 instance for real
* Identify and fully resolve configuration/code issues that surfaced in production
* Ensure both roles (Member, Admin) can log in reliably

### Tasks to be carried out this week:

| No. | Task | Start Date | Estimated Time |
| :-: | :--- | :-: | :-: |
| **1** | - Deploy the Cognito/TOTP code to EC2, run `composer install`, run the DB migration | `20/07/2026` | 1 day |
| **2** | - Fix OAuth configuration errors: `invalid_scope`, Google missing from the App Client's Identity Provider list | `21/07/2026` | 2 days |
| **3** | - Fix a runtime error: HTTP 500 when creating a new user from Cognito (SQL bind parameter mismatch) | `23/07/2026` | 1 day |
| **4** | - Test the full flow: Member logging in with Google, Admin logging in with Google + TOTP, and the legacy username/password login | `24/07/2026` | 2 days |
| **5** | - Sync the code changes made directly on EC2 back to GitHub, and clean up security (rotate the RDS password, reset the Google Client Secret) | `26/07/2026` | 1 day |

### Week 7 Achievements:

* The Google login flow for both Member and Admin now runs reliably in production; Admin must pass the TOTP step before being granted access to `/admin`.
* Identified and fixed 3 main issues:
  * **`invalid_scope`**: the App Client was missing the `profile` scope in its OAuth Connect scopes.
  * **Generic HTTP 400 from Cognito**: used the AWS CLI (via CloudShell) to read the App Client configuration directly, discovering that `SupportedIdentityProviders` only contained `COGNITO` and was missing `Google` — fixed via `update-user-pool-client`.
  * **HTTP 500 when creating a new user**: the auto username-generation function ran a nested SELECT query in the middle of binding parameters for an INSERT statement, corrupting the bound data (because the `Database` class shares a single prepared statement across the whole app) — fixed by computing the username before starting the bind sequence.
* Learned to use the AWS CLI/CloudShell to debug directly instead of relying only on the Console UI — much more effective when the Console shows a vague error.

### Limitations:

* Debugging still relies on manually reading Apache logs (`error_log`), since CloudWatch Log streaming for Cognito hasn't been enabled (it belongs to the paid Plus plan) — an accepted trade-off for the learning stage.
* A few deployments got out of sync between EC2 and GitHub (a forgotten file push, a misnamed file) — a clear sign of the need for automated CI/CD in future projects.