# Switch

### A dead man's switch for encrypted files

If you stop checking in, Switch emails an encrypted file to someone you choose.
The file is encrypted in your browser before it is sent to the service.

> [!IMPORTANT]
> This repository currently contains documentation and a license only. The
> application source, database schema, migrations, and deployment configuration
> described below are not included, so this checkout cannot be deployed as-is.

## Contents

- [Features](#features)
- [How it works](#how-it-works)
- [Who it's for](#who-its-for)
- [Security model](#security-model)
- [Limitations](#limitations)
- [Deployment notes](#deployment-notes)
- [Repository contents](#repository-contents)

## Features

- **Set the check-in cadence to anything, not just whole days.** The setup
  page now has one-click presets &mdash; every hour, every 6 hours, every day,
  every week, every month &mdash; plus a custom amount + unit if you want
  something else (e.g. "every 10 minutes" for testing, or "every 4 months").
  It shows a plain-English summary of exactly when the warning and the
  release will happen before you submit.
- **A password-protected cancel button.** `/cancel?token=...` permanently
  disarms a switch. It requires a *separate* cancel password (set when the
  switch is created) — different from the check-in link, so someone who
  only has the check-in link can't shut the whole thing off.
- **A simple passphrase mode**, so nobody has to understand PGP. At setup,
  the owner picks a passphrase (e.g. a few random words), and the file is
  locked with it using standard AES-256 encryption in the browser. The
  owner tells the recipient the passphrase separately — by phone, in
  person, whatever channel isn't the same email carrying the file. PGP
  mode still exists for anyone who already uses it (toggle on the setup
  page).
- **A no-install page for the recipient to open the file:** `/decrypt`.
  They upload the file they received, type the passphrase, and it decrypts
  and downloads in their browser. No PGP tools, no command line.

## How it works

1. **Create a switch.** Choose a check-in interval, grace period, encryption
   mode, and recipient. Save the separate check-in and cancel links.
2. **Check in on schedule.** Each check-in resets the timer for the next one.
3. **Respond to a warning.** If a check-in is missed, the service sends a
   warning and starts the configured grace period.
4. **Release or cancel.** The warning email includes a password-protected
   cancel link. If the switch remains armed after the grace period, the
   encrypted file is emailed to the recipient.
5. **Open the file.** The recipient decrypts the attachment with the
   passphrase in passphrase mode, or with their private key in PGP mode.

## Who it's for

**Deploying the app is still a technical, one-time task** — creating a
Vercel project, a database, an email-sending account, and setting
environment variables. That part genuinely needs someone comfortable with
those tools; there's no way around it for a real, independently-running
backend (see the earlier explanation of why a browser-only tool can't do
this reliably).

**Once deployed, using it is not technical at all.** Creating a
switch, checking in, cancelling, and opening a received file are all
plain forms with plain-language instructions — built so a person who's
never heard of encryption can do all of it correctly.

## Security model

- **The server never has plaintext or a decryption key.** In passphrase
  mode, the passphrase never touches the server — only the resulting
  locked file does, and it's useless without the passphrase. In PGP mode,
  only the recipient's private key can open it.
- **A single missed check-in doesn't trigger anything.** There's a
  warning email with a grace period first, and a way to cancel from that
  same email.
- **Cancelling requires a password nobody else has**, separate from the
  check-in link — so it can't be turned off just because someone found
  the check-in link.

## Limitations

- **True sender anonymity.** Your Vercel account, domain, and Resend
  account are tied to billing information that can be subpoenaed. This
  hides the file's contents from the server operator and from anyone who
  breaches the database — it does not hide *that a switch exists* from a
  legally motivated investigator. Ask if you need to go further (Tor
  relay, anonymous hosting payment).
- **Multi-user accounts.** This is built for one owner. `/setup` is
  protected by a single shared "site passphrase" (`SETUP_SECRET`), not
  real logins. Fine for personal use.
- **Recovering a lost passphrase or cancel password.** By design, nothing
  is recoverable — that's what makes them real security, not just a
  formality.

## Deployment notes

These notes describe the intended setup, but cannot be followed from this
checkout until the application and configuration files are added. In
particular, the referenced `schema.sql`, migrations, `.env.example`, and
`vercel.json` are not present here.

1. **Create a Vercel project** from this folder.
2. **Add Vercel Postgres** (Storage tab → Create Database → Postgres).
   Sets `POSTGRES_URL` automatically.
3. **Run `schema.sql`** in the Postgres query console for a brand-new
   install. If you already deployed an earlier version:
   - Only had the very first version (no cancel password)? Run
     `migrate_v2.sql`, then `migrate_v3.sql`.
   - Already had the cancel-password version (`interval_days` /
     `grace_days` columns)? Just run `migrate_v3.sql` — it upgrades those
     to minute-based `interval_minutes` / `grace_minutes` columns without
     losing your existing switches.
4. **Create a Resend account**, verify a sending domain, get an API key.
5. **Set environment variables** (Project Settings → Environment
   Variables), per `.env.example`:
   - `RESEND_API_KEY`
   - `FROM_EMAIL` — address on your verified Resend domain
   - `BASE_URL` — your deployed URL
   - `SETUP_SECRET` — random string
     (`node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`)
   - `CRON_SECRET` — another random string, same command
6. **Protect the cron route.** Project Settings → Cron Jobs → enable
   "Secure your Cron Jobs" → set it to the same value as `CRON_SECRET`.
7. **About the cron schedule.** The deployment notes describe `/api/cron`
  running hourly (`0 * * * *`), which is what lets an "every hour" switch get
   checked hourly. **Vercel's free Hobby plan only runs cron jobs once a
   day**, no matter what schedule you put in `vercel.json` — on Hobby, an
   hourly or every-6-hours switch will only ever be checked once a day, so
   the release could be delayed by up to a day past its real deadline.
   Two ways around that on Hobby:
   - Use daily-or-longer intervals only (the safest, zero-extra-setup option), or
   - Keep the short interval, and add a free external pinger (e.g.
     [cron-job.org](https://cron-job.org)) that calls
     `GET https://your-app.vercel.app/api/cron` every hour with header
     `Authorization: Bearer <your CRON_SECRET>`. The endpoint is safe to
     call as often as you like — it only ever acts on switches that are
     actually due.
   On a Vercel Pro plan, the hourly schedule in `vercel.json` just works
   as-is.
8. **Deploy.** `vercel --prod`.
9. **Create your first switch** at `/setup`. Pick a check-in cadence from
   the presets (or use "Custom…" for something like every 10 minutes, just
   for testing) and save both links it gives you — the check-in link and
   the cancel link — somewhere durable.
10. **Test before trusting it.** Use the "Custom…" option to set a tiny
    interval (e.g. 5 minutes) and a tiny grace period, backdate
    `last_checkin_at` in the DB if you don't want to wait, manually call
    `/api/cron` with the right `Authorization` header, confirm the warning
    email arrives, then the release email, then confirm `/decrypt` actually
    opens the file. Also test `/cancel` with the wrong password (should
    fail) and the right one (should permanently stop it). Reset your test
    row after — or just create a fresh switch with your real cadence.

## Repository contents

- `README.md` — project overview and deployment notes
- `LICENSE` — MIT License
