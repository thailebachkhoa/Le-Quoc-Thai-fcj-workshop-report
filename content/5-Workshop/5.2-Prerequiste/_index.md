---
title: "2. Basic Infrastructure Setup: EC2 + RDS"
weight: 2
date: 2026-08-05
draft: false
---

## 2.1. Creating a VPC (If Not Already Present in Your Account)

A new AWS account might not have a VPC in the selected region. Go to **EC2 → Launch Instance**; if a yellow warning saying "no VPCs in this region" appears, click the link **"create a new default VPC"** — AWS will automatically create a VPC + Subnets in each Availability Zone + an Internet Gateway, requiring no additional configuration.

## 2.2. Launching an EC2 Instance

- AMI: Ubuntu (latest available LTS version)
- Instance type: `t3.micro` (Free Tier)
- Storage: 20GB gp3
- Security Group: SSH (port 22) restricted to **"My IP"**, HTTP (80) and HTTPS (443) set to **Anywhere**
- Key pair: Create a new key pair, download the `.pem` file, and save it securely (can only be downloaded once)

## 2.3. Connecting via SSH

```bash
chmod 400 your-key-file.pem
ssh -i your-key-file.pem ubuntu@<Public-IP>
```

## 2.4. Installing Required Software

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2
sudo apt install -y php libapache2-mod-php php-mysql php-mbstring php-curl php-xml php-zip php-gd php-fileinfo
sudo apt install -y ffmpeg
sudo apt install -y git unzip mysql-client-core
```

## 2.5. Deploying Code to Server

```bash
cd /var/www
sudo git clone [https://github.com/thailebachkhoa/FCAJ-Intern-Project.git](https://github.com/thailebachkhoa/FCAJ-Intern-Project.git) plantify
sudo chown -R www-data:www-data /var/www/plantify
sudo chmod -R 775 /var/www/plantify/storage
```

## 2.6. Configuring Apache Virtual Host

```bash
sudo nano /etc/apache2/sites-available/plantify.conf
```

```apache
<VirtualHost *:80>
    ServerName <Public-IP-or-domain>
    DocumentRoot /var/www/plantify/public

    <Directory /var/www/plantify/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/plantify_error.log
    CustomLog ${APACHE_LOG_DIR}/plantify_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite plantify.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Important note**: `DocumentRoot` must point to `public/`, not the project root — if pointed incorrectly, the `.env` file (containing database passwords) will be directly exposed via URL.

## 2.7. Creating an RDS MySQL Instance

- Engine: MySQL, Template: **Free tier**
- Instance: `db.t4g.micro`
- Public access: **No**
- VPC security group: Create new
- Initial database name: `plantify` (if left blank, you must manually execute `CREATE DATABASE` later)

## 2.8. Connecting EC2 ↔ RDS

In the RDS Console, under the **Connectivity & security** tab → **Connected compute resources → Set up EC2 connection** → select the correct EC2 instance — AWS automatically adds the required Security Group rule allowing the EC2 instance to connect to RDS, eliminating the need to manually modify Security Groups.

## 2.9. Importing Schema

```bash
mysql -h <rds-endpoint> -P 3306 -u admin -p
```
```sql
CREATE DATABASE plantify CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
exit;
```
```bash
mysql -h <rds-endpoint> -P 3306 -u admin -p plantify < /var/www/plantify/database/migrations/schema.sql
```

## 2.10. Updating `.env` and Testing

```bash
sudo nano /var/www/plantify/.env
```
```
DB_HOST=<rds-endpoint>
DB_PORT=3306
DB_DATABASE=plantify
DB_USERNAME=admin
DB_PASSWORD=<actual-password>
```

Open `http://<Public-IP>` in your browser — the Plantify homepage should render normally.

## Common Issues

| Issue | Cause | Solution |
|---|---|---|
| SSH freezes or does not respond | Security Group is still whitelisting an old IP while your network IP has changed | Check your current IP at `checkip.amazonaws.com`, then update the SSH rule to "My IP" |
| `chmod 400` warns "Permissions too open" despite being executed | The `.pem` file is located on a Windows drive (`/mnt/d/...` via WSL) — `chmod` does not take real effect on NTFS | Copy the key file to `~/.ssh/` (actual Linux filesystem) and rerun `chmod 400` |
| `mysql: command not found` | MySQL client is not installed on the EC2 instance (distinct from the `php-mysql` PHP extension) | `sudo apt install -y mysql-client-core` |
| Importing schema returns "No such file or directory" | Incorrect path — the file is inside `database/migrations/`, not `database/` | Verify using `ls -la database/migrations` before importing |
| RDS has no `plantify` database despite creation | Left "Initial database name" blank when creating RDS | Manually run `CREATE DATABASE plantify ...` via `mysql` CLI |