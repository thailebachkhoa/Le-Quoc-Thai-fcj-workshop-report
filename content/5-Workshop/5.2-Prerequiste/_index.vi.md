---
title: "2. Dựng hạ tầng cơ bản: EC2 + RDS"
weight: 2
date: 2026-08-05
draft: false
---

## 2.1. Tạo VPC (nếu tài khoản chưa có sẵn)

Tài khoản AWS mới có thể chưa có VPC nào trong region đang chọn. Vào **EC2 → Launch Instance**, nếu thấy cảnh báo vàng "no VPCs in this region", bấm link **"create a new default VPC"** — AWS tự tạo VPC + Subnet ở mỗi Availability Zone + Internet Gateway, không cần cấu hình gì thêm.

## 2.2. Tạo EC2 instance

- AMI: Ubuntu (bản LTS mới nhất có sẵn)
- Instance type: `t3.micro` (Free Tier)
- Storage: 20GB gp3
- Security Group: SSH (port 22) giới hạn **"My IP"**, HTTP (80) và HTTPS (443) mở **Anywhere**
- Key pair: tạo mới, tải về `.pem`, lưu cẩn thận (chỉ tải được 1 lần)

## 2.3. Kết nối SSH

```bash
chmod 400 ten-file-key.pem
ssh -i ten-file-key.pem ubuntu@<Public-IP>
```

## 2.4. Cài phần mềm cần thiết

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2
sudo apt install -y php libapache2-mod-php php-mysql php-mbstring php-curl php-xml php-zip php-gd php-fileinfo
sudo apt install -y ffmpeg
sudo apt install -y git unzip mysql-client-core
```

## 2.5. Đưa code lên server

```bash
cd /var/www
sudo git clone https://github.com/thailebachkhoa/FCAJ-Intern-Project.git plantify
sudo chown -R www-data:www-data /var/www/plantify
sudo chmod -R 775 /var/www/plantify/storage
```

## 2.6. Cấu hình Apache Virtual Host

```bash
sudo nano /etc/apache2/sites-available/plantify.conf
```

```apache
<VirtualHost *:80>
    ServerName <Public-IP-hoặc-domain>
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

**Lưu ý quan trọng**: `DocumentRoot` phải trỏ vào `public/`, không phải root project — nếu trỏ nhầm, file `.env` (chứa mật khẩu database) sẽ bị lộ trực tiếp qua URL.

## 2.7. Tạo RDS MySQL

- Engine: MySQL, Template: **Free tier**
- Instance: `db.t4g.micro`
- Public access: **No**
- VPC security group: tạo mới
- Initial database name: `plantify` (nếu bỏ trống, phải tự `CREATE DATABASE` sau)

## 2.8. Kết nối EC2 ↔ RDS

Trong RDS Console, tab **Connectivity & security → Connected compute resources → Set up EC2 connection** → chọn đúng instance EC2 — AWS tự động thêm rule Security Group cho phép EC2 kết nối vào RDS, không cần tự sửa Security Group tay.

## 2.9. Import schema

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

## 2.10. Cập nhật `.env` và test

```bash
sudo nano /var/www/plantify/.env
```
```
DB_HOST=<rds-endpoint>
DB_PORT=3306
DB_DATABASE=plantify
DB_USERNAME=admin
DB_PASSWORD=<mật khẩu thật>
```

Mở `http://<Public-IP>` trên trình duyệt — phải thấy trang chủ Plantify hiện lên bình thường.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| SSH bị treo, không phản hồi | Security Group vẫn whitelist IP cũ trong khi IP mạng đã đổi | Kiểm tra IP hiện tại tại `checkip.amazonaws.com`, sửa lại rule SSH thành "My IP" |
| `chmod 400` báo "Permissions too open" dù đã chạy | File `.pem` đang nằm trên ổ Windows (`/mnt/d/...` qua WSL) — `chmod` không có hiệu lực thật trên NTFS | Copy file key vào `~/.ssh/` (filesystem Linux thật) rồi `chmod 400` lại |
| `mysql: command not found` | Chưa cài MySQL client trên EC2 (khác với PHP extension `php-mysql`) | `sudo apt install -y mysql-client-core` |
| Import schema báo "No such file or directory" | Sai đường dẫn — file nằm trong `database/migrations/`, không phải `database/` | Kiểm tra bằng `ls -la database/migrations` trước khi import |
| RDS không có database `plantify` dù đã tạo | Bỏ trống "Initial database name" lúc tạo RDS | Tự `CREATE DATABASE plantify ...` qua `mysql` CLI |