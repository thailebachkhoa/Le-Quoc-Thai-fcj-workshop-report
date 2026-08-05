---

title: "4. Domain, HTTPS, and Elastic IP"
weight: 4
date: 2026-08-05
draft: false
------------

## 4.1. Assigning an Elastic IP

An EC2 instance’s default public IP address changes every time the instance is stopped and started, which becomes inconvenient because SSH configurations and DNS records must be updated repeatedly.

To assign a permanent public IP address:

**EC2 → Network & Security → Elastic IPs → Allocate Elastic IP address**

Then:

**Actions → Associate Elastic IP address**

Select the correct EC2 instance.

Using an Elastic IP ensures that the server remains reachable through the same IP address even after a restart.

> **Cost note:** Since **February 1, 2024**, AWS charges for all public IPv4 addresses (approximately **$0.005/hour**), including Elastic IPs attached to running instances. The EC2 Free Tier includes **750 hours of public IPv4 usage per month during the first 12 months**, which is sufficient if you use only a single Elastic IP.

## 4.2. Configuring a free domain with DuckDNS

Register a free subdomain at **duckdns.org** and point it to the Elastic IP created in the previous step.

After the DNS record has propagated, update the Apache Virtual Host configuration so that the `ServerName` matches the new domain.

Example:

```apache
ServerName your-subdomain.duckdns.org
```

## 4.3. Enabling HTTPS with Let’s Encrypt

Install Certbot and the Apache integration:

<CodeBlock language=
