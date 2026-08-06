---
title: "Amazon Cognito, Explained Properly: User Pools, Hosted UI, Federation, and What the Token Actually Contains"
date: 2026-08-04
draft: false
tags: ["aws", "cognito", "identity", "authentication"]
description: "A ground-up explanation of Amazon Cognito — what it actually is, its two very different halves, how federated login works under the hood, and what's really inside the JWT it hands back to your app."
---

Amazon Cognito is one of those AWS services that gets used constantly and understood loosely. Most tutorials show you which buttons to click to get Google login working, without explaining what Cognito is actually doing in between. This post is the explanation that would have saved a lot of confusion before building authentication for the Plantify Co project.

## First: Cognito Is Two Different Services Wearing One Name

This is the single most common source of confusion. "Amazon Cognito" bundles two products that solve different problems:

- **User Pools** — a user directory and authentication service. It answers *"who is this person, and how do they prove it?"* It can store users itself, or broker identity from external providers (Google, Facebook, corporate SAML/OIDC IdPs). This is what handles login screens, passwords, MFA, and federated sign-in.
- **Identity Pools** — a way to hand out **temporary AWS credentials** to your app's users, so a mobile or web app can call AWS services (like S3) directly, without your backend acting as a middleman. It answers *"now that I know who you are, what AWS permissions should you get?"*

A huge number of projects — including Plantify Co — only ever need **User Pools**. If you're not letting end users' browsers call AWS APIs directly, you can safely ignore Identity Pools entirely and a lot of Cognito's reputation for being "confusing" disappears.

## Inside a User Pool: The Building Blocks

- **User Pool** — the directory itself. Holds user records, their attributes (email, name...), and settings like password policy or MFA requirements.
- **App Client** — represents *an application* that's allowed to authenticate against the pool. A single User Pool can have multiple App Clients (e.g. a web app and a mobile app), each with its own allowed callback URLs, OAuth scopes, and allowed Identity Providers. This separation is why forgetting to enable an Identity Provider on the *App Client* — even after adding it to the User Pool — is such a common setup mistake: the pool level and client level are configured independently.
- **Domain (Hosted UI)** — a ready-made login page hosted by AWS at `<prefix>.auth.<region>.amazoncognito.com`, so you don't have to build your own login form to start the OAuth2 flow.
- **Identity Providers** — external services (Google, Facebook, an SSO provider...) that Cognito can delegate authentication to, instead of managing a password itself.
- **Groups** — a simple way to tag users with roles (e.g. `Admin`, `Member`); group membership shows up directly inside the issued token.

## How Federated Login Actually Flows

When a user signs in via Google through Cognito, four parties are involved, and it's worth being precise about who talks to whom:

1. **Your app → Cognito**: redirects the browser to Cognito's Hosted UI, optionally specifying `identity_provider=Google` to skip straight to Google instead of showing Cognito's own picker.
2. **Cognito → Google**: redirects the browser again, this time to Google's actual consent screen. Your app is not involved in this step at all.
3. **Google → Cognito**: after the user approves, Google sends an authorization code back — but to *Cognito's* redirect URI, not your app's. Cognito then calls Google's token endpoint server-to-server to exchange that code for Google's tokens, and either creates or matches a user record in the User Pool.
4. **Cognito → your app**: redirects the browser one final time, to *your app's* callback URL, with a Cognito-issued authorization code. Your app exchanges this for Cognito's own tokens.

The key thing this reveals: your application never sees Google's tokens or the user's Google password at any point. It only ever receives tokens **issued by Cognito**, which is precisely why verifying those tokens correctly matters — your app is trusting Cognito's word, not Google's directly.

## What's Actually Inside the Token

Cognito issues a JSON Web Token (JWT) — a signed, three-part string (`header.payload.signature`). The payload is a set of claims, and for a federated login it typically includes:

```json
{
  "sub": "a1b2c3d4-...",
  "iss": "https://cognito-idp.<region>.amazonaws.com/<user-pool-id>",
  "aud": "<app-client-id>",
  "token_use": "id",
  "email": "user@gmail.com",
  "name": "User Name",
  "cognito:groups": ["Admin"],
  "exp": 1735689600
}
```

A correct verification checks more than "does this string decode into JSON" — it must confirm: the **signature** is valid against Cognito's public keys (JWKS), `iss` matches your exact User Pool, `aud` matches your exact App Client, and `token_use` is the token type you expect (`id` vs `access` tokens serve different purposes and shouldn't be treated interchangeably). Skipping any of these checks means your app would accept a token forged for a *different* app client or a *different* pool — decoding a JWT without verifying it is functionally the same as not checking authentication at all.

## What Cognito Is Genuinely Good At (and What It Isn't)

**Strong fit**: standardizing OAuth2/OIDC so you don't hand-roll it per provider, centralizing user/group management, and giving you a Hosted UI so a working login screen exists on day one.

**Not a fit**: anything requiring per-user or per-group logic layered *into* the authentication step itself — Cognito's MFA, for instance, applies at the User Pool level, not per group, and doesn't compose cleanly with federated logins. Read [Blog 1](../3.1-Blog1/) if you also want to see how Plantify Co used an IAM Role instead of Cognito Identity Pools to let EC2 talk to S3 — a good example of picking the right AWS identity tool for the right layer of a system, rather than reaching for the same one everywhere.

## Takeaway

Cognito is less mysterious once you stop treating it as one black box and start treating it as: a user directory (User Pools) that can delegate to external providers, wrapped in a standard OAuth2 flow, issuing a JWT that your app is responsible for verifying properly. Most Cognito confusion — including a fair share of the debugging done on this project — traces back to blurring those pieces together instead of reasoning about them one at a time.