# Upgrade guide

Your version is shown in **Settings › System health** (and in the sidebar footer). Each release lists its changes in `CHANGELOG.md`.

## Steps

1. **Back up** the database (phpMyAdmin *Export*, or `mysqldump`) and these files: `.env`, `public/uploads/`, `storage/app/private/`.
2. **Upload the new release over the old files.** Keep your `.env`, `public/uploads/` and `storage/` (the release ZIP contains neither `.env` nor uploads, so extracting it over the old folder is safe).
3. **Run the upgrade command**:

   ```bash
   php artisan toran:upgrade
   ```

   It puts the site in maintenance mode, runs new database migrations, clears and rebuilds the caches, records the new version, and brings the site back.

   **No SSH?** Use your host's *Terminal* or *Run PHP command* tool, or a one-off cron job (`cd /path/to/toran && php artisan toran:upgrade`) that you delete afterwards.
4. Open **Settings › System health** and check the new version and that everything is green.

If something goes wrong, restore the backup of the files and the database.

## By hand

`toran:upgrade` is the same as:

```bash
php artisan down
php artisan migrate --force
php artisan optimize:clear
php artisan optimize        # production only
php artisan up
```

## Customised copies

If you changed code (themes, modules), keep your changes in version control and merge the new release into your branch before uploading. Changes made in Settings (branding, languages per user, messaging keys) live in the database and are not affected. If you changed `resources/`, rebuild the assets with `npm run build` after merging.

## From a copy installed before 1.0.0

Copies installed with `php artisan toran:install` before the web installer existed have no `storage/app/installed.lock`, so they would open the installer. `php artisan toran:upgrade` creates the lock when an account exists.
