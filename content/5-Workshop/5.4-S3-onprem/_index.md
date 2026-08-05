---
title: "4. Domain, HTTPS, Elastic IP"
weight: 4
date: 2026-08-05
draft: false
---

## 4.1. Assigning an Elastic IP

EC2's default Public IP changes every time the instance is Stopped/Started — which is inconvenient as SSH and domain settings must be continuously updated. Assign a static IP address:

**EC2 → Network & Security → Elastic IPs → Allocate Elastic IP address** → **Actions → Associate Elastic IP address** → Select the correct instance.

> Cost Note: As of February 1, 2024, AWS charges for all public IPv4 addresses (~$0.005/hour), even when attached to a running instance. EC2 Free Tier includes 750 hours of IPv4 per month for the first 12 months — sufficient to cover costs if only 1 address is used.

## 4.2. Free Domain via DuckDNS

Register at [duckdns.org](https://www.duckdns.org) and point your subdomain to the newly created Elastic IP. Update `ServerName` in the Apache Virtual Host configuration to match the new domain.

## 4.3. HTTPS via Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d <subdomain>.duckdns.org
```

Select the option to **redirect all HTTP traffic to HTTPS** when prompted by Certbot. The SSL certificate auto-renews automatically, requiring no further manual action.

## Common Issues

| Issue | Cause | Solution |
|---|---|---|
| Unable to SSH using the old IP after a Stop/Start | Elastic IP not attached; the Public IP has changed | Check the new Public IP on the EC2 Console, or attach an Elastic IP to prevent recurrence |
| Certbot fails to issue a certificate | Domain does not point to the correct IP yet, or port 80 is not open in the Security Group | Verify that `ping <domain>` returns the correct IP, and confirm that the Security Group allows HTTP/HTTPS traffic |