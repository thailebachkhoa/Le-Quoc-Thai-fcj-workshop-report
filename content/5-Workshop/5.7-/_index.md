---

title: "7. Fixing mobile UI issues and removing unused code"
weight: 7
date: 2026-08-05
draft: false
------------

## 7.1. Horizontal overflow on mobile

On a 390px mobile screen, the first characters of the hero section text were clipped. The root cause was that Bootstrap’s `.row.g-5` (24px gutter) exceeded the default `.container` horizontal padding (12px), causing the row to overflow horizontally. Because the parent section used `overflow: hidden`, the overflowing content was cut off.

**Fix 1 — Global overflow protection**

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

**Fix 2 — Use responsive gutters**

Keep generous spacing on desktop while reducing it on smaller screens.

```html
<!-- Before -->
<div class="row g-5 align-items-end">

<!-- After -->
<div class="row gy-5 gx-3 gx-lg-5 align-items-end">
```

This change was applied to all **seven affected locations** across three files:

* `home.php`
* `faq.php`
* `product-detail.php`

## 7.2. Removing the orphaned About page

After moving the old About page content into sections on the homepage, the related admin components (`admin/pages.php` and `public/api/upload-video.php`) became hidden routes. They were no longer accessible through the UI, but could still be reached by manually entering their URLs.

The cleanup process included:

1. Delete `app/Views/admin/pages.php` and `public/api/upload-video.php`.
2. Remove the `AdminController::pages()` and `AdminController::save_pages()` methods.
3. Rename the content keys from `about.process_*` / `about.map_*` to `home.process_*` / `home.map_*`, and manage them through the `page_home()` editor instead of the removed About editor.
4. Remove `'pages'` from the route array in `Sidebar.php`.

## Common issues

| Issue                                                                        | Cause                                                                                     | Solution                                                                             |
| ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| The old layout still appears after updating the CSS                          | The browser is using a cached stylesheet                                                  | Perform a hard refresh (`Ctrl+Shift+R`) or enable **Disable cache** in DevTools      |
| `git pull` on EC2 returns **Permission denied**                              | The project directory is owned by `www-data`, so the `ubuntu` user cannot write to `.git` | Run `sudo git pull`, then restore ownership with `sudo chown -R www-data:www-data`   |
| A view file was deleted but the corresponding controller method still exists | The cleanup was incomplete                                                                | Always remove the view and the controller method that references it at the same time |
