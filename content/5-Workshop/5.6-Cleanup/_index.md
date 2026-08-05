---

title: "6. Replacing authentication with Cognito, Google OAuth, and TOTP"
weight: 6
date: 2026-08-05
draft: false
------------

## 6.1. Authentication architecture

Instead of implementing the entire login and password system manually (which carries a higher security risk if implemented incorrectly), the application was migrated to **Amazon Cognito** as the identity provider, with **Google** as the external Identity Provider (IdP). Users authenticate with their real Google accounts rather than receiving OTP codes by email.

Authentication flow:

```text
User clicks "Sign in with Google"
    → Cognito Hosted UI
        → Redirect to Google (accounts.google.com)
            → User signs in with a Google account
        → Google redirects back to Cognito (/oauth2/idpresponse)
    → Cognito issues an id_token (JWT) and redirects to the application (/auth/callback)
    → PHP verifies the JWT and reads cognito:groups to determine whether the user is an Admin or Member
    → If the user is an Admin, a TOTP verification step is required before accessing /admin
```

A critical limitation is that Cognito **does not allow built-in MFA to be combined with passwordless/federated sign-in for the same account**. Therefore, Admin MFA was implemented **at the application layer**, completely independent of Cognito’s built-in MFA features.

## 6.2. Creating the Google OAuth client

1. Open **Google Cloud Console** and create a new project (a personal Google account is recommended to avoid organization policy restrictions).
2. Go to **APIs & Services → OAuth consent screen**, configure the application, and **publish the app**. In Testing mode, Google limits the application to a small number of test users.
3. Go to **Credentials → Create OAuth Client ID → Web application**, and configure the redirect URI:

```text
https://<cognito-domain>.auth.<region>.amazoncognito.com/oauth2/idpresponse
```

## 6.3. Configuring Cognito

1. **User Pools → Create user pool**

   * Application type: **Traditional web application**
2. **Sign-in experience**

   * Federated identity provider sign-in
   * Add identity provider: **Google**
   * Enter the Google Client ID and Client Secret
   * Authorized scopes: `profile email openid`
3. **App integration → Domain**

   * Create a Cognito domain for the Hosted UI
4. **App clients → Create app client**

   * Allowed callback URLs:
     `https://<domain>/auth/callback`
   * Identity providers:
     **Enable Google**
     (this is the step most commonly overlooked because Cognito user pool is selected by default)
   * OAuth grant type:
     **Authorization code grant**
   * OpenID Connect scopes:
     `email`, `openid`, `profile`

## 6.4. Database migration

```sql
ALTER TABLE users
    ADD COLUMN cognito_sub VARCHAR(100) NULL UNIQUE AFTER id,
    ADD COLUMN totp_secret VARCHAR(64) NULL AFTER role,
    MODIFY COLUMN password VARCHAR(255) NULL;
```

The `password` column is now nullable because users authenticating through Google no longer require a local password.

## 6.5. PHP implementation

Main components:

* `app/Core/Cognito.php`

  * OAuth redirect
  * Exchange authorization code for tokens
  * JWT verification using `firebase/php-jwt`
* `app/Core/Totp.php`

  * Generates and verifies 6-digit TOTP codes
  * Implemented from scratch without external TOTP libraries
* `app/Views/auth/totp.php`

  * Admin TOTP verification screen
* `.env` additions:

```text
COGNITO_REGION
COGNITO_USER_POOL_ID
COGNITO_APP_CLIENT_ID
COGNITO_APP_CLIENT_SECRET
COGNITO_DOMAIN
COGNITO_REDIRECT_URI
```

## 6.6. Creating the first Admin account

The Cognito Hosted UI allows users to self-register only as **Members**. There is intentionally no self-service option to become an Admin.

The first administrator must be promoted manually using the AWS CLI:

```bash
aws cognito-idp admin-add-user-to-group \
  --user-pool-id <user-pool-id> \
  --username <email> \
  --group-name Admin
```

## Common issues

| Issue                                                   | Cause                                                                                                                   | Solution                                                                                     |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `composer install` is blocked by a security warning     | An old version of `firebase/php-jwt` contains a known vulnerability                                                     | Upgrade to version 7 or later                                                                |
| `invalid_scope` or `invalid_request` during login       | The `profile` scope was not enabled in the App Client                                                                   | Add `profile` to the OpenID Connect scopes                                                   |
| A generic HTTP 400 error even though scopes are correct | The App Client does not allow **Google** as an identity provider (`SupportedIdentityProviders` contains only `COGNITO`) | Open **Login pages → Edit** and enable **Google** under Identity providers                   |
| HTTP 500 error during the callback process              | The `Database` class is implemented as a singleton, and nested SQL queries caused PDO parameter binding conflicts       | Avoid nested queries on the same singleton PDO connection; execute them sequentially         |
| `login.php` still shows the old version on the server   | The updated file was not pushed to GitHub before running `git pull` on EC2                                              | Check `git status` and `git log` before pulling to ensure all local changes have been pushed |
