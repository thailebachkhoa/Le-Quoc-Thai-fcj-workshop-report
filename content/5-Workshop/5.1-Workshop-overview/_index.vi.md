---
title: "1. Rà soát và vá bảo mật code"
weight: 1
date: 2026-08-05
draft: false
---

Trước khi đưa bất kỳ ứng dụng nào lên môi trường công khai, cần rà soát lại các lỗ hổng bảo mật cơ bản. Đây là bước làm trước tiên, trước cả khi đụng tới AWS Console.

## 1.1. Thêm CSRF protection toàn diện

Ứng dụng ban đầu không có cơ chế chống CSRF (Cross-Site Request Forgery) ở bất kỳ action ghi/xóa dữ liệu nào — cho phép kẻ tấn công dựng 1 trang giả, khi nạn nhân (đang đăng nhập) mở trang đó, request bị gửi thay mặt nạn nhân mà họ không biết.

Xây dựng class dùng chung:

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
            die('Yêu cầu không hợp lệ.');
        }
    }
}
```

Áp dụng `Csrf::verify()` trong constructor của **7 khu vực**: `AdminController`, `AuthController`, `CartController`, `ShopController`, `NewsController`, `DashboardController`, và endpoint độc lập `upload-video.php`. Mọi action ghi/xóa dữ liệu đổi từ GET (`<a href>`) sang POST (`<form>`).

## 1.2. Ẩn thông tin lỗi hệ thống

Trước:
```php
} catch (PDOException $e) {
    die("Database Connection Error: " . $e->getMessage());
}
```

Sau:
```php
} catch (PDOException $e) {
    error_log('DB Connection Error: ' . $e->getMessage());
    die('Hệ thống đang bảo trì, vui lòng quay lại sau.');
}
```

Tắt `display_errors` trong `php.ini` ở môi trường production.

## 1.3. Các bug nhỏ khác đã sửa

- `?>?>` lặp đôi trong `header.php`/`dashboard/index.php` — in ký tự thừa ở đầu mọi trang.
- `composer.json` sai case (`bootstrap.php` → `Bootstrap.php`) — không lỗi trên Windows nhưng lỗi trên Linux (case-sensitive filesystem).
- `schema.sql` dư dấu phẩy trước `ON DUPLICATE KEY UPDATE` — lỗi cú pháp khi import MySQL thật.
- `FileController::render()` dùng `strpos()` thay vì `realpath()` để chặn path traversal — nâng cấp lên `realpath()` cho chắc chắn.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| CSRF token không gửi lên dù đã thêm `csrf_field()` | Đặt `csrf_field()` sai vị trí — nhét vào giữa các thuộc tính của thẻ `<form ...>` thay vì đặt sau dấu `>` đóng thẻ | `csrf_field()` phải là 1 phần tử con của form, đứng sau dấu `>` |
| Toàn bộ form POST trong admin bị 403 sau khi thêm `Csrf::verify()` | Quên thêm `<?= csrf_field() ?>` vào 1 vài form cũ chưa cập nhật | Rà soát lại toàn bộ form POST, thêm token vào từng cái |