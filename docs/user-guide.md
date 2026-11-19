# User guide

Short steps for each part of Toran. What you can see depends on your role:

| Role | Can use |
|---|---|
| **Admin** | Everything, including Settings › Branding, Messaging and System health, and undoing check-ins |
| **Planner** | Everything except account settings; sees money (shagun totals, vendors, budget) |
| **Staff** | The check-in desk only (and their own profile, password and language) |

Your language is chosen in **Settings › Preferences** (English, हिन्दी or ગુજરાતી).

## Dashboard

The first page after signing in: countdown to the main event, guests and RSVP counts by side, today's shagun, the event timeline, tasks for today, vendor payments due (admins and planners) and the check-in card. The bell (top right) shows payment reminders, new RSVPs and today's open tasks. Switch celebrations with the selector at the top left.

## Events

1. **Events › New celebration** for a wedding, janoi, seemant or any function.
2. Add each event (Mehndi, Sangeet, Lagna, Reception…) with date, time, venue, map link and dress code; mark one as the main event.
3. Event status: draft, ready, done.

## Guests

1. **Guests › Add guest**, choosing a new or an existing family, side (bride, groom, common), phone, diet and the events they are invited to.
2. **Import** from Excel or CSV: upload, match the columns, check the preview, import. Download the sample sheet first if you are unsure.
3. Filter by side, event, RSVP, city or tag; select rows for bulk actions (invite to an event, tag, delete); **Export** to Excel.
4. Click a family to open its drawer: members, invitations, RSVP link, WhatsApp opt-out and the change history.

## RSVP

1. Each family has a private RSVP link (copy it from the family drawer or the RSVP page). Guests answer per event, with headcount and food preference; no login.
2. The RSVP page shows every family's answer per event; you can change an answer by hand.
3. **Send reminder** sends a WhatsApp reminder to families who haven't replied.
4. Families who said yes see their **entry pass** (a QR code) on their RSVP page.

## Invitations

1. **Invitations** › pick a sample design (Royal, Minimal, Pastel) or upload your own background.
2. In the editor, drag text layers, change font, size and colour, and insert placeholders like `{family_name}`. **Add QR code** prints each family's check-in code on their invitation.
3. **Preview** shows exactly what a family will receive.
4. **Create invitations** for an event: one image per invited family, in the family's language. **Download all (ZIP)** or send them on WhatsApp.

## WhatsApp

1. **WhatsApp › New send**: invitations for an event (with the images) or RSVP reminders.
2. With click-to-chat, use **Send one by one** and press send in your WhatsApp for each family. With Meta or Twilio, messages go out automatically.
3. The send log shows each family's status (sent, delivered, read, failed, skipped, opted out). **Retry** a failed one.

## Seating

1. **Seating** › choose an event › **Add table** (round or rectangular, seats, label).
2. Drag tables to arrange the room. Drag a family or a guest from *Not seated* onto a table (or tap the guest, then the table).
3. **Auto-seat** keeps families together. **Export** CSV or PDF for the venue.
4. Table numbers appear on the check-in screen.

## Shagun ledger

1. **Shagun Ledger › Add shagun** (or press **N**): giver, amount, mode (cash, UPI, cheque, gift), event and who received it. The dialog stays open for the next envelope.
2. Totals by event, side and mode; filter by date, event, side or mode.
3. **Excel** and **PDF** export the account book with a running balance.

## Vendors

1. **Vendors › Add vendor**: name, category (Caterer, Decorator, Photographer, Venue, Music/DJ, Makeup, Priest, Transport, Other), contact person, phone, email, notes and the **total agreed**.
2. On the vendor's page, **Add installment** for each payment: amount, due date, event, mode, and tick *This is the advance* for the advance. Attach the receipt (PDF or photo).
3. When you pay, click **Mark paid** (or fill in *Paid on*). Overdue installments are highlighted in red.
4. **Documents**: upload the quotation and the contract (PDF or image, up to 10 MB). They open through a private link that expires after 15 minutes.

## Budget

1. **Budget › Set budget**: the amount you plan to spend per category.
2. The chart shows **planned vs spent** for each category: the bar is what has been paid, the dark tick is the plan, red means over budget. Switch to **Table** for the numbers.
3. The tiles show the total plan, spent, remaining and overdue amounts. *Show overdue payments* filters the schedule.
4. **Add payment** also records expenses without a vendor (return gifts, hotel rooms…).
5. **Excel** and **PDF** export the summary and the full payment schedule.

Reminders: admins and planners get a bell notification 3 days before an installment is due and again when it becomes overdue (needs the cron job).

## Check-in (event day)

1. Open **Check-in** on a phone or tablet (staff accounts land here directly) and pick the event. **Full screen** hides everything else.
2. **Scan**: point the camera at the QR code on the guest's invitation or RSVP page. The family appears with how many people are expected, their table and food preference.
3. Set **how many people arrived** with − and +, then **Check in**.
4. No QR code? Use **Search** (name or phone). Someone not on the list? **Walk-in**: name, phone, side and number of people.
5. A family already checked in shows a clear warning with the time and who checked them in. Admins can **Undo** a mistaken check-in.
6. The counters (checked in, expected, walk-ins) update live and include other desks every 30 seconds.
7. **No network?** Keep going: check-ins are saved on the device ("Offline… N waiting to sync") and sent automatically when the connection returns. Keep the page open until the count reaches zero.

The camera needs HTTPS. If the camera is refused or unavailable, use Search.

## Settings

- **Preferences**: your language (and, for admins, the account's default language).
- **Profile / Password**: your name, email and password.
- **Branding** (admins): app name, tagline, logo, favicon, colours, sign-in page headline and text, footer text.
- **Messaging** (admins): WhatsApp method, keys, message texts per language, throttling.
- **System health** (admins): version, server checks, cron and queue status, upgrade steps.
