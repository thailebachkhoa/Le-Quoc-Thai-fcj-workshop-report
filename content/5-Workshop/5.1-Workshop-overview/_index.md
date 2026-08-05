---
title: "1. Code Security Review and Patching"
weight: 1
date: 2026-08-05
draft: false
---

Before deploying any application to a public environment, basic security vulnerabilities must be reviewed and patched. This is the essential first step, prior to interacting with the AWS Console.

## 1.1. Implementing Comprehensive CSRF Protection

The initial application lacked a CSRF (Cross-Site Request Forgery) defense mechanism on any data write/delete actions — allowing an attacker to create a malicious page where, if a logged-in victim opens it, requests are sent on their behalf without their knowledge.

Creating a shared class:

```php
class Csrf
{
    public static function token()
    {
        if (empty($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }
        return $_SESSION['csrf_token'];
    }

    public static function field()
    {
        return '<input type="hidden" name="csrf_token" value="' . self::token() . '">';
    }

    public static function verify()
    {
        $sent = $_POST['csrf_token'] ?? '';
        $real = $_SESSION['csrf_token'] ?? '';
        if ($real === '' || !hash_equals($real, $sent)) {
            http_response_code(403);
            die('Invalid request.');
        }
    }
}
```

Apply `Csrf::verify()` in the constructor of **7 areas**: `AdminController`, `AuthController`, `CartController`, `ShopController`, `NewsController`, `DashboardController`, and the standalone endpoint `upload-video.php`. All data write/delete actions were converted from GET (`<a href>`) to POST (`<form>`).

## 1.2. Hiding System Error Details

Before:
```php
} catch (PDOException $e) {
    die("Database Connection Error: " . $e->getMessage());
}
```

After:
```php
} catch (PDOException $e) {
    error_log('DB Connection Error: ' . $e->getMessage());
    die('The system is undergoing maintenance, please try again later.');
}
```

Disable `display_errors` in `php.ini` in the production environment.

## 1.3. Other Fixed Minor Bugs

- Duplicate `?>?>` in `header.php`/`dashboard/index.php` — rendered extra characters at the top of every page.
- Incorrect casing in `composer.json` (`bootstrap.php` → `Bootstrap.php`) — no error on Windows, but caused failures on Linux due to its case-sensitive filesystem.
- `schema.sql` contained an extra trailing comma before `ON DUPLICATE KEY UPDATE` — causing a syntax error when importing into actual MySQL.
- `FileController::render()` used `strpos()` instead of `realpath()` to prevent path traversal — upgraded to `realpath()` for robust verification.

## Common Issues

| Issue | Cause | Solution |
|---|---|---|
| CSRF token is not sent despite adding `csrf_field()` | Incorrect placement of `csrf_field()` — placed inside the attributes of the `<form ...>` tag instead of after the closing `>` tag | `csrf_field()` must be a child element of the form, placed after the closing `>` bracket |
| All admin POST forms return 403 after adding `Csrf::verify()` | Forgot to add `<?= csrf_field() ?>` to a few old forms that were not updated | Audit all POST forms and add the token to each one |