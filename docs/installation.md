# Installation

Toran runs on ordinary PHP hosting. Pick one of three paths:

- [Shared hosting](#shared-hosting-cpanel-plesk-hostinger) (cPanel, Plesk, Hostinger…): upload, open `/install`, add one cron line.
- [VPS](#vps-ubuntu--debian) (Ubuntu or Debian with Nginx or Apache).
- [Docker](#docker-optional) (optional; app, MySQL, queue worker and scheduler in containers).

## Requirements

| | |
|---|---|
| PHP | 8.2 or newer with `pdo_mysql`, `mbstring`, `intl`, `fileinfo`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `curl`, `zip`; the `proc_open` function must not be disabled (the cron scheduler uses it to start its tasks) |
| Images | GD with FreeType (on almost every host) **or** Imagick |
| Database | MySQL 5.7+ or MariaDB 10.3+, an empty database and a user for it |
| Cron | One line, every minute (reminders, WhatsApp sends, invitation images) |
| HTTPS | Needed for the camera scanner on the check-in screen (browsers only allow the camera on HTTPS). Everything else works on HTTP. |

No Node.js, Redis, Composer, SSH or symlinks are needed on the server: the release ZIP already contains `vendor/` and the built assets in `public/build/`. PHP's `proc_open` must be enabled, because Laravel's scheduler starts each cron task as a process; a few shared hosts list it under `disable_functions`, so ask your host to allow it. The installer's server check warns when it is off.

The installer checks all of this for you on its first page, and **Settings › System health** shows the same checks after installation.

## The web installer

Once the files are on the server, open `https://your-domain/install`:

0. **Setup code.** The installer first asks for the code in `storage/app/install-token` (a file such as `K7QF-3MZT-XW9A`, created on the first visit). Open it in your hosting file manager or over FTP and type the code. Only someone with access to the server's files can read it, so nobody who stumbles on a freshly uploaded copy can install it before you. The file is deleted when installation finishes. If `storage/` isn't writable yet, the page says so and lists the folders to fix.
1. **Server check.** PHP version, extensions and writable folders (`storage/`, `bootstrap/cache/`, `public/uploads/` and `.env`). Fix anything marked ✕, then *Check again*.
2. **Database.** Host, port, database name, user and password. *Test and save* connects first; only a working connection is written to `.env`.
3. **Tables.** Creates the tables (`php artisan migrate`). Safe to run again.
4. **Account.** App name, your business (or family) name, the first admin (name, email, password), default language (English unless you choose Hindi or Gujarati), timezone, and optionally the **demo data** (a sample wedding with 300 guests, RSVPs, shagun, seating, vendors and a budget).

When you finish, the installer:

- writes `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL`, `APP_NAME` and `APP_LOCALE` to `.env`;
- creates `storage/app/installed.lock`, after which `/install` returns 404 for good, and deletes the setup code;
- shows the cron line to add.

On a fresh upload with no `.env`, the first visit copies `.env.example` to `.env` and generates `APP_KEY` (see `bootstrap/first-run.php`). If the project folder isn't writable, create `.env` yourself from `.env.example` and set `APP_KEY` with `php artisan key:generate`.

To run the installer again, delete `storage/app/installed.lock` (your data stays; step 4 then just closes the installer if an account exists). A new setup code is created on the next visit.

**Command-line alternative:** `php artisan migrate --force && php artisan toran:install` asks the same questions in the terminal and also closes the web installer.

## Shared hosting (cPanel, Plesk, Hostinger)

1. Download the release ZIP (`toran-1.1.0.zip`) and upload it with the file manager, then extract it. You get a folder like `toran/` with `app/`, `public/`, `vendor/`…
2. **Point the domain at `public/`.** In cPanel: *Domains › your domain › Document Root* → `toran/public`. In Plesk: *Hosting Settings › Document root*.

   **If your host won't let you change the document root**, put the project in the domain's root folder (for example `public_html/`) as it is. The `.htaccess` in the project root sends every request into `public/`, so `.env`, `vendor/` and `storage/` are never reachable from the web. This works for a domain or a subdomain, not for a sub-folder like `example.com/toran`.
3. **PHP version and extensions.** In *Select PHP Version* (or *MultiPHP Manager*) choose PHP 8.2 or newer and tick `intl`, `fileinfo`, `zip` and `gd` if they aren't on.
4. **Database.** In *MySQL Databases*, create a database and a user, and add the user to the database with all privileges.
5. Open `https://your-domain/install` and follow the steps.
6. **Cron.** In *Cron Jobs*, add (every minute):

   ```
   * * * * * cd /home/USER/toran && /usr/local/bin/php artisan schedule:run >> /dev/null 2>&1
   ```

   Use the PHP path your host shows (for example `/opt/cpanel/ea-php82/root/usr/bin/php`). The cron line also works the queue for up to 50 seconds a minute, so no separate worker is needed. **Settings › System health** shows when the scheduler last ran.
7. **HTTPS.** Turn on the free certificate (AutoSSL / Let's Encrypt) so the check-in camera works.

Folder permissions: `storage/`, `bootstrap/cache/` and `public/uploads/` must be writable by PHP (usually `755` for folders on suPHP/LSAPI hosts, `775` elsewhere).

## VPS (Ubuntu / Debian)

```bash
sudo apt install nginx mariadb-server php8.3-fpm php8.3-{mysql,mbstring,intl,xml,curl,zip,gd} unzip
sudo mysql -e "CREATE DATABASE toran CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; CREATE USER 'toran'@'localhost' IDENTIFIED BY 'choose-a-password'; GRANT ALL ON toran.* TO 'toran'@'localhost';"
sudo unzip toran-1.1.0.zip -d /var/www/
sudo chown -R www-data:www-data /var/www/toran/storage /var/www/toran/bootstrap/cache /var/www/toran/public/uploads
sudo chown www-data:www-data /var/www/toran && sudo chmod 775 /var/www/toran   # lets the installer create .env
```

Nginx site (`/etc/nginx/sites-available/toran`):

```nginx
server {
    listen 80;
    server_name events.example.com;
    root /var/www/toran/public;
    index index.php;
    client_max_body_size 20M;

    location / { try_files $uri $uri/ /index.php?$query_string; }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
    location ~ ^/uploads/.*\.php$ { deny all; }
    location ~ /\.(?!well-known) { deny all; }
}
```

Enable it, add HTTPS (`sudo certbot --nginx`), then open `/install`. After installing, set the folder back to `755`.

**Background jobs on a VPS.** Keep the cron line (`crontab -u www-data -e`), and run a real queue worker under Supervisor instead of the cron-based one:

```ini
; /etc/supervisor/conf.d/toran.conf
[program:toran-queue]
command=php /var/www/toran/artisan queue:work --sleep=1 --max-time=3600
user=www-data
autostart=true
autorestart=true
stopwaitsecs=60
```

and set `QUEUE_WORKER_VIA_SCHEDULER=false` in `.env`.

Apache works too: point `DocumentRoot` at `public/` and enable `mod_rewrite`; `public/.htaccess` does the rest.

## Docker (optional)

`Dockerfile` and `docker-compose.yml` in the project root run four containers: **app** (PHP-FPM + Nginx), **mysql**, **queue** (`queue:work`) and **scheduler** (`schedule:work`).

```bash
cp .env.example .env
# set DB_HOST=mysql, DB_DATABASE=toran, DB_USERNAME=toran, DB_PASSWORD=..., and the same values in docker-compose.yml
docker compose up -d --build
```

Open http://localhost:8080/install. Uploads, logs and the database live in named volumes (`toran-storage`, `toran-uploads`, `toran-mysql`), so `docker compose down` keeps them. Put a TLS proxy (Caddy, Traefik, Nginx) in front for HTTPS.

The Docker files are provided as a starting point and were not part of the release testing (see the FAQ).

## After installing

- Sign in, then open **Settings › System health** and check everything is green, including *The scheduler is running*.
- **Settings › Branding**: your name, logo, favicon, colours, sign-in text and footer.
- **Settings › Messaging**: how WhatsApp messages are sent (see [configuration.md](configuration.md)).
- Add your team (see the FAQ: users are created by an admin; there is no public sign-up).
