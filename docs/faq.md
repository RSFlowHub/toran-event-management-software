# FAQ and troubleshooting

### The site shows "500 Server Error" or a blank page
Check `storage/logs/laravel.log`. The usual causes: `storage/` or `bootstrap/cache/` not writable, a missing PHP extension (see Settings › System health or `/install`), or wrong database details in `.env`. To see the error in the browser for a moment, set `APP_DEBUG=true` in `.env`, reload, then set it back to `false`.

### Every page sends me to /install
The lock file `storage/app/installed.lock` is missing. Finish the installer, or, for a copy installed from the command line before 1.0.0, run `php artisan toran:upgrade`.

### I need to run the installer again
Delete `storage/app/installed.lock`. Your data is kept; if an account exists, the last step just closes the installer again. You'll need the new setup code from `storage/app/install-token`.

### The installer asks for a setup code
Open `storage/app/install-token` in your hosting file manager (or over FTP) and type the code inside. It protects a freshly uploaded copy from being installed by someone else. If the page says the code couldn't be created, make `storage/` writable (permission 775) and reload.

### "The scheduler is not running" in System health
The cron line is missing, uses the wrong PHP path, or points at the wrong folder. Copy the exact line from System health into your host's Cron Jobs (every minute). Until it runs, invitation images and WhatsApp messages stay queued and payment reminders aren't created. If the cron job does run but nothing happens, check that PHP's `proc_open` isn't disabled on your host (Settings › System health shows it).

### Invitation images stay at 0 %
Same as above: the queue is worked by the cron job. Locally, run `php artisan queue:work`.

### The check-in camera doesn't start
Browsers allow the camera only on HTTPS (or `localhost`). Turn on SSL for the domain. If the browser asked and the answer was "Block", allow the camera in the site settings. Search by name or phone always works.

### Check-ins were made without internet. Are they lost?
No. They are kept on that device and sent when it is online again; the badge shows how many are waiting. Don't clear the browser's data on that device before the badge disappears. A family checked in at two desks while offline is recorded once; the second desk gets a "already checked in" notice when it syncs.

### A guest's QR code says "Not an Toran check-in code"
Codes are signed with `APP_KEY`. If `APP_KEY` changed after the invitations went out, old codes stop working: search by name instead. Never change `APP_KEY` on a live site.

### Gujarati or Hindi letters look wrong in invitation images
With GD, joined letters (conjuncts like શ્રી) don't join. Install Imagick with raqm/Pango on the server (or ask your host); Toran uses it automatically.

### WhatsApp messages fail
Use **Send test** in Settings › Messaging. Meta needs an approved template outside the 24-hour window; Twilio needs an approved Content template. The send log shows the provider's reason.

### How do I add team members?
There is no public sign-up and no team screen yet. An admin can create users with:

```bash
php artisan tinker
>>> app(App\Support\Tenancy\TenantContext::class)->run(App\Models\Tenant::first(), fn () => App\Models\User::create(['name' => 'Rohan', 'email' => 'rohan@example.com', 'password' => bcrypt('a-strong-password'), 'role' => 'staff']));
```

Roles: `admin`, `planner`, `staff` (check-in only).

### How do I reset a demo server?
`php artisan toran:demo-reset` erases **all data and uploads** and reloads the sample wedding (logins `admin@toran.test` / `password`). It refuses to run in production without `--force`. Only use it on demo servers.

For a **public demo** that customers try themselves:

1. Install with the demo data (`php artisan toran:install --demo --force`) on its own database.
2. Set `DEMO_MODE=true` in `.env`. The sign-in page then lists the sample logins (click one to sign in), every page shows a demo banner, each visitor's language choice is kept in their own browser session (not on the shared login), and passwords, the sign-in email, password reset, Branding, Messaging, account settings and deleting the celebration are locked. Everything else can be tried.
3. Keep Settings › Messaging on click-to-chat or log-only. Never put real Meta or Twilio keys on a demo server.
4. Reload the sample wedding every hour with a cron line next to the scheduler:

   ```
   0 * * * * cd /path/to/toran && php artisan toran:demo-reset --force >> /dev/null 2>&1
   ```

### Does Toran contact any outside server?
No. There is no licence check, telemetry or tracking. Outside requests happen only for features you turn on: WhatsApp providers (Meta, Twilio) when you send messages. Hindi, Marathi and Gujarati fonts are served from your own server.

### Are the Docker files tested?
They were written for this release but not run in its test environment. Treat them as a starting point and test them on your setup.

### Where are files stored, and what should I back up?
The database, `public/uploads/` (logos, invitation images, backgrounds, fonts) and `storage/app/private/` (vendor documents and receipts). See [configuration.md](configuration.md#files-and-storage).
