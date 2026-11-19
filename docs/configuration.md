# Configuration

Most settings are made in the app (**Settings**). A few server settings live in `.env`; after editing `.env` on a server that caches its config, run `php artisan optimize:clear`.

## `.env` reference

| Key | Default | What it does |
|---|---|---|
| `APP_NAME` | `Toran` | Fallback product name (Settings › Branding overrides it per account). |
| `APP_URL` | | The public address, with `https://`. Used in RSVP links, invitation image links and webhooks. **Set it correctly.** |
| `APP_ENV` / `APP_DEBUG` | `local` / `true` in `.env.example`; the web installer sets `production` / `false` | Keep debug off in production: error pages would show internals. If you set up from the command line, change both yourself. |
| `APP_LOCALE` | `en` | Default language for new accounts (`en`, `hi`, `gu`). |
| `DB_*` | | Database connection (written by the installer). |
| `SESSION_DRIVER` | `database` | Sessions in the database (`file` also works). |
| `QUEUE_CONNECTION` | `database` | Background jobs in the `jobs` table. `redis` is supported; `sync` runs jobs during the request (slow, no retries). |
| `QUEUE_WORKER_VIA_SCHEDULER` | `true` | The cron line also works the queue. Set `false` when you run `queue:work` under Supervisor. |
| `INVITATION_RENDERER` | (auto) | `gd` or `imagick`; empty = Imagick when installed, else GD. |
| `RSVP_REMINDER_COOLDOWN_HOURS` | `12` | Minimum gap before the same family gets another RSVP reminder. |
| `TENANCY_MODE` | `single` | `single`: one account per install. `domain`: hosted, one account per domain. |
| `LOCALE_FONT_CSS` | `/fonts/locale/{slug}.css` (bundled) | Stylesheet for the Hindi/Gujarati UI fonts. Set it to a Google Fonts URL (`{family}` placeholder) if you prefer smaller downloads from Google. |
| `MAIL_*` | `log` | Mail for password resets. Set your SMTP details. |
| `TORAN_INSTALLED` | (empty) | Leave empty: `storage/app/installed.lock` decides. |
| `DEMO_MODE` | `false` | For a public demo server only. Shows the sample logins on the sign-in page (one click signs in) and a demo banner, keeps each visitor's language choice in their session, and locks the changes that would spoil the demo for the next visitor: passwords, profile (sign-in email), password reset, Branding, Messaging, account settings and deleting a celebration. See the FAQ on demo servers. |

## Cron and the queue

One cron line, every minute, runs Laravel's scheduler (`routes/console.php`):

```
* * * * * cd /path/to/toran && php artisan schedule:run >> /dev/null 2>&1
```

| Scheduled task | When | What |
|---|---|---|
| `queue:work --stop-when-empty --max-time=50` | every minute | Works background jobs (invitation images, WhatsApp sends). Off when `QUEUE_WORKER_VIA_SCHEDULER=false`. |
| heartbeat | every minute | Records the time, shown in Settings › System health. |
| `toran:payment-reminders` | hourly | Bell notifications for vendor payments due within 3 days or overdue (once each), for admins and planners. |
| `queue:prune-failed` | daily | Removes failed jobs older than 7 days. |

**Settings › System health** shows when the scheduler last ran, how many jobs are waiting and how many failed. If it says the scheduler isn't running, the cron line is missing or uses the wrong PHP path.

## WhatsApp providers

Toran only sends through official routes (unofficial bulk tools break WhatsApp's terms and get numbers banned). Choose the method in **Settings › Messaging** (admins):

| Method | How it works | You need |
|---|---|---|
| **Click-to-chat** (default) | A `wa.me` link per family with the message filled in; *Send one by one* opens each chat and you press send in your own WhatsApp. | Nothing |
| **Meta Cloud API** | Sent automatically from your WhatsApp Business number; delivered/read receipts by webhook. | Meta business account, phone number ID, permanent access token, app secret, approved template |
| **Twilio** | Sent through Twilio; delivery receipts by status callback. | Account SID, auth token, WhatsApp sender, approved Content template |
| **Log only** | Writes messages to the log instead of sending. | Nothing (testing) |

### Meta (WhatsApp Cloud API)

1. In Meta for Developers, create an app with the WhatsApp product and add your business phone number.
2. Create a system user with a **permanent access token** (`whatsapp_business_messaging`).
3. In WhatsApp Manager, get a message template approved, for example `wedding_invitation` with body variables (family name, RSVP link) and an optional image header.
4. In Settings › Messaging choose **Meta** and enter the phone number ID, access token, app secret, a verify token of your choice, the template name and language, and the template variables in order (`family_name,rsvp_link`).
5. In the Meta app's WhatsApp › Configuration, set the callback URL to `https://your-domain/webhooks/whatsapp/meta`, the same verify token, and subscribe to `messages`.

Without a template, plain text is sent, which WhatsApp delivers only within 24 hours of the guest writing to you. Invitation images must be reachable on a public HTTPS URL, so set `APP_URL`.

### Twilio

1. Enable a WhatsApp sender in the Twilio Console (or the sandbox for testing).
2. Create and get approval for a WhatsApp **Content template**; note its `HX…` SID.
3. In Settings › Messaging choose **Twilio** and enter the account SID, auth token, sender (`+14155238886`), the Content SID and its variables.
4. Status callbacks are requested on every message (`https://your-domain/webhooks/whatsapp/twilio`); set the same URL as the sender's incoming-message webhook so STOP replies opt families out.

Use **Send test** in Settings › Messaging before a real campaign. Provider keys are stored encrypted with `APP_KEY` (never in `.env`, never logged, never sent back to the browser). **Keep `APP_KEY`**: changing it makes saved keys unreadable and invalidates printed check-in QR codes.

Throttling (messages per minute) and retry attempts are set in Settings › Messaging. Temporary errors are retried after 1, 5 and 15 minutes.

## Files and storage

| Where | What | Served |
|---|---|---|
| `public/uploads/branding/` | Logos and favicons | Publicly (needed in the page) |
| `public/uploads/invitations/`, `invitation-backgrounds/`, `fonts/` | Generated invitation images, design backgrounds, uploaded fonts | Publicly, under random names (images are sent to guests) |
| `storage/app/private/{account}/` | Vendor quotations and contracts, payment receipts | **Never directly**: only via a 15-minute signed link, to signed-in admins and planners |
| `storage/app/private/imports/` | Guest import sheets during an import | No |
| `storage/logs/` | Laravel log | No |

Back up the database **and** `public/uploads/` and `storage/app/private/`.

## Mail

Password-reset emails need working mail. Set `MAIL_MAILER=smtp` and `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_FROM_ADDRESS`. With `MAIL_MAILER=log` the reset link is written to `storage/logs/laravel.log` instead.
