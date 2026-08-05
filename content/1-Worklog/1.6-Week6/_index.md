---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Replace traditional username/password login with Google sign-in, using Amazon Cognito as the authentication broker
* Enforce two-factor authentication (TOTP - Google Authenticator) specifically for Admin accounts
* Update the code and database to support the new authentication flow

### Tasks to be carried out this week:

| No. | Task | Start Date | Estimated Time |
| :-: | :--- | :-: | :-: |
| **1** | - Create the Cognito User Pool + App Client, create a Cognito Domain (Hosted UI) | `13/07/2026` | 1 day |
| **2** | - Create an OAuth Client in Google Cloud Console, add Google as an Identity Provider in Cognito, configure attribute mapping (email, name) | `14/07/2026` | 1 day |
| **3** | - Write `Cognito.php` (exchange the authorization code for a token, verify the JWT signature via JWKS) and `Totp.php` (generate/verify TOTP codes per RFC 6238, with no external dependency) | `15/07/2026` | 2 days |
| **4** | - Update `AuthController`, `User` model: add the `google()`/`callback()`/`totp()` flow, sync the Admin/Member role from the Cognito Group into the existing `role` column | `17/07/2026` | 2 days |
| **5** | - Write a migration adding `cognito_sub` and `totp_secret` columns to the `users` table; add the `auth/totp.php` view (QR scan / code entry screen) | `19/07/2026` | 1 day |

### Week 6 Achievements:

* Add an additional authencation step: both User and Admin sign in through a single "Sign in with Google" button; PHP reads the Cognito Group in the token to distinguish roles; Admin accounts are additionally required to enter a TOTP code before a real session is granted.
* Kept the legacy username/password login form working in parallel (not removed) for easier rollback, while applying the same mandatory TOTP rule for Admin on both login paths — there is no shortcut that bypasses this protection layer.
* All related code (Cognito, TOTP, DB migration) is written and ready to be deployed to the real EC2 instance the following week.

### Limitations:

* The AWS Console / Google Cloud Console configuration was only done based on theory and written notes, not yet tested on the production environment — a fair number of configuration issues were expected to surface once deployed for real (confirmed in Week 7).