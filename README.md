# ShopWave — Full-Stack E-commerce (PHP + MySQL + JavaScript)

Storefront + admin panel. Vanilla JS front end, PHP 8.1+ JSON API (PDO), MySQL 5.7+/8 or MariaDB 10.3+.
No framework or build step: upload and run.

## Features
**Storefront:** product grid, full-text + prefix search (debounced, stale-request aborting), category / price / in-stock filters, sorting, pagination, product detail, persistent cart, server-side price re-verification, GST + shipping rules, register/login, checkout, order history, order cancellation (auto-restock).
**Admin (`/admin.html`):** revenue dashboard, order management with status workflow, product CRUD with validated image upload, soft-hide products, category creation.
**Engineering:** all SQL via prepared statements; composite + FULLTEXT indexes for listing/search; transactional checkout using `SELECT … FOR UPDATE` (no overselling under concurrency, verified); price/name snapshots on order lines; CSRF tokens; bcrypt/argon hashing with rehash; session fixation protection; login rate limiting; upload MIME + image validation with random filenames and a no-execute uploads dir; escaped output everywhere (XSS); CSP/security headers; HTTP caching on static assets.

## ⚠️ Important: don't double-click index.html
The store needs PHP + MySQL, so it must be served by a web server (Docker / Apache / `php -S` below). If you open `public/index.html`
straight from disk (file://), you only get a **preview mode** with sample products so you can see the design — login, checkout and admin need the server.
Always unzip the **whole** folder first (the page needs `assets/`).

## Deploy option A — Docker (fastest)
```bash
cp .env.example .env          # set DB_PASS, ADMIN_EMAIL, ADMIN_PASSWORD
docker compose up -d --build  # app on http://localhost:8080 ; schema, demo catalog and admin created automatically
```
Put it behind HTTPS (Caddy / Nginx / Cloudflare / your host's proxy). Session cookies auto-set `Secure` when HTTPS or `X-Forwarded-Proto: https` is seen.

## Deploy option B — Shared hosting / VPS (Apache or Nginx)
1. Upload the project; **point the web root at `public/`** (keep `src/`, `database/`, `scripts/` outside it). Nginx sample: `nginx.conf.example`.
2. Create a MySQL database + user, set env vars (or edit `src/config.php` defaults): `DB_HOST DB_NAME DB_USER DB_PASS`.
3. Run once: `ADMIN_EMAIL=you@domain.com ADMIN_PASSWORD='Strong-Pass-1' php scripts/install.php`
   (no SSH? import `database/schema.sql` + `database/seed.sql` in phpMyAdmin and create the admin via the installer locally against the remote DB.)
4. Make `public/uploads/` writable by the PHP user. Enable `mod_rewrite` (Apache). Use HTTPS.
5. Visit `/` and `/admin.html`. Use `--no-seed` on install if you don't want the demo catalog.

## Local run without Apache
```bash
export DB_USER=... DB_PASS=... ; php scripts/install.php
PHP_CLI_SERVER_WORKERS=8 php -S 127.0.0.1:8080 -t public tests/router.php
```

## Tests
`BASE=http://127.0.0.1:8080 ADMIN_EMAIL=… ADMIN_PASSWORD=… tests/smoke.sh` — 39 HTTP checks: catalog/search, CSRF, auth, pricing/GST, stock, cancel/restock, **10 parallel buyers vs 3 units**, admin CRUD, malicious upload.
Manual UAT script: `tests/UAT_CHECKLIST.md`.

## Before go-live checklist
- [ ] Change the admin password; `APP_DEBUG=0`
- [ ] HTTPS enabled; DB user has only the app database
- [ ] Replace demo products / tax rate / shipping in `src/config.php` (`tax_rate`, `ship_fee`, `free_ship_over`)
- [ ] Back up DB (`mysqldump`) on a schedule
- [ ] **Payments:** "online" currently records the method only (invoice flow). To take card/UPI payments integrate a gateway (Razorpay/Stripe) in `h_order_create` and mark orders `paid` from its webhook.
- [ ] Email notifications (order confirmation) are not included — add via PHPMailer/SMTP if needed.

## Layout
```
public/        web root: index.html, admin.html, assets/, api/index.php (router), uploads/
src/           bootstrap.php (db, csrf, auth, pricing) · handlers_public.php · handlers_admin.php · config.php
database/      schema.sql (indexes) · seed.sql
scripts/       install.php
tests/         smoke.sh · router.php · UAT_CHECKLIST.md
```
