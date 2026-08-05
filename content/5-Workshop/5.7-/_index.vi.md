---
title: "7. Vá lỗi UI mobile và dọn dẹp code thừa"
weight: 7
date: 2026-08-05
draft: false
---

## 7.1. Lỗi tràn ngang trên mobile

Trên màn hình điện thoại (390px), chữ ở phần hero bị mất ký tự đầu dòng. Nguyên nhân: Bootstrap `.row.g-5` (gutter 24px) vượt quá padding mặc định của `.container` (12px) — phần dư tràn ra ngoài và bị section cha (`overflow: hidden`) cắt cụt.

**Sửa 1 — Lưới an toàn chung:**
```css
html {
  overflow-x: hidden;
}
body {
  margin: 0;
  overflow-x: hidden;
  ...
}
```

**Sửa 2 — Đổi gutter thành responsive**, giữ khoảng cách đẹp ở desktop, thu nhỏ ở mobile:
```html
<!-- Trước -->
<div class="row g-5 align-items-end">
<!-- Sau -->
<div class="row gy-5 gx-3 gx-lg-5 align-items-end">
```

Áp dụng cho toàn bộ 7 vị trí trong 3 file (`home.php`, `faq.php`, `product-detail.php`).

## 7.2. Dọn dẹp "About page" mồ côi

Sau khi tách nội dung trang About cũ vào các section trong trang chủ, phần quản trị (`admin/pages.php`, `public/api/upload-video.php`) trở thành route ẩn — không hiện trên UI nhưng vẫn truy cập được nếu gõ tay URL.

Việc cần làm:
1. Xóa `app/Views/admin/pages.php` và `public/api/upload-video.php`.
2. Xóa method `AdminController::pages()` và `save_pages()`.
3. Đổi key nội dung `about.process_*`/`about.map_*` → `home.process_*`/`home.map_*`, chuyển sang quản lý trong editor `page_home()` thay vì editor "about" đã xóa.
4. Bỏ `'pages'` khỏi mảng route trong `Sidebar.php`.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| Sửa CSS xong nhưng ảnh chụp vẫn thấy lỗi cũ | Trình duyệt cache bản CSS cũ | Hard refresh (`Ctrl+Shift+R`) hoặc tick "Disable cache" trong DevTools |
| `git pull` trên EC2 báo "Permission denied" | Thư mục thuộc sở hữu `www-data`, user `ubuntu` không có quyền ghi vào `.git` | Chạy `sudo git pull`, sau đó `sudo chown -R www-data:www-data` lại |
| Xóa file view nhưng quên xóa controller method tương ứng | Dọn dẹp không đồng bộ giữa View và Controller | Luôn kiểm tra chéo: xóa View → tìm Controller method gọi View đó → xóa cùng lúc |