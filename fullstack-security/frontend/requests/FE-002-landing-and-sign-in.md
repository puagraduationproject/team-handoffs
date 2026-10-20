---
id: FE-002
title: Landing page and the sign-up / log-in pages
from: backend
to: frontend
cc: [security]
status: open
priority: high
created: 2026-10-21
updated: 2026-10-21
author: backend
---

# Landing page and the sign-up / log-in pages

## Summary

A landing page with the sign-up box on it, its own log-in page, and the short
sign-up steps (Google, or email then a code then a password). The backend for
all of it is built and live on the test address; the pages there now are plain
placeholders for you to replace.

## Context

- Design: `technical-structure.md` section 7.6 (sign up and log in) and section
  17.2, window 15 (landing page).
- Backend routes, inputs, outputs and errors:
  `app-backend/platform/auth/README.md`. Error codes:
  `app-backend/contracts/errors/README.md`.
- **Reference for the flow and the structure only:** [claude.com](https://claude.com)
  and its log-in page. The sign-up box sits on the landing page itself, next
  to a preview of the product, and the log-in page is its own page with the
  same box.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. The reference site is for
how the pages work and what is on them, not for how they look. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Pages

| Page | Path | What is on it |
|------|------|---------------|
| Landing | `/` | Top bar, the hero with the sign-up box and a product preview, the sections below, the footer |
| Log in | `/login` | The same sign-up box, alone on its own page |
| The steps | `/login` (same page, step by step) | Code, set a password, password, Google account |
| My account | `/account` | What the person has (already built as a plain page; restyle it) |

Logged-in people skip the landing page and go to the app (sessions exist
now; see My account below).

### Landing page

1. **Top bar:** the project's name or logo, **Log in** (goes to `/login`) and
   **Get started** (scrolls to the sign-up box, or goes to `/login`)
2. **Hero:**
   - A short headline and one line under it saying what the product is: one AI
     that runs a small business (chats, orders, stock, cash), for coffee shops
     and small markets first. Wording is yours; a suggestion is in Texts below
   - **The sign-up box** (see below)
   - **A product preview** beside it: the AI chat at work, for example a
     morning briefing as cards ("Milk runs out tomorrow", "3 orders waiting",
     "Yesterday's cash: 2,450 EGP"). Static is fine; moving is a bonus
3. **Sections below** (order and look are yours):
   - What it does for a coffee shop and for a small market (3 to 6 points)
   - How it works, in three steps: sign up, set up the business, talk to the AI
   - Questions and answers (FAQ)
4. **Footer:** the name, the team, the university, links to the privacy and
   terms pages (placeholders for now)

### The sign-up box

The same box on the landing page and on `/login`:

```
[ Continue with Google ]
─────────── or ───────────
[ Continue with email ]
By continuing, you agree to the Terms of Service and Privacy Policy.
```

- **Continue with Google:** a normal link to `GET /api/auth/google/start`. The
  backend sends the person to Google and back to `/account#ticket=...`
- **Continue with email:** on the landing page it goes to `/login` with the
  email field open; on `/login` it shows the email field in the box
- After the email: `POST /api/auth/start { email }`. The answer says the next
  step: `code`, `password` or `google`

### The steps after the email

Only two steps to sign up with email:

**New account (`next: "code"`)**

1. **Code:** "We sent a 6-digit code to {email}." A box for 6 digits, **Send a
   new code** (locked for 60 seconds, with a countdown), "Use another email".
   `POST /api/auth/code/verify { email, code }` gives `ticket` and
   `needs_password: true`
2. **Set a password:** password and the same again, at least 8 characters.
   `POST /api/auth/password/set { ticket, password }`. Then `/account`

**Account with a password (`next: "password"`)**

- Password box, **Log in**, and **Email me a code instead**
  (`POST /api/auth/code/send { email }`, then the code step; the code logs
  them in without a new password).
  `POST /api/auth/password/login { email, password }`. Then `/account`

**Google account (`next: "google"`)**

- "{email} signs in with Google." **Continue with Google**, and **Email me a
  code instead**

**Coming back from Google**

- `/account#ticket=...`: read the ticket, remove it from the address bar, then
  `POST /api/auth/ticket { ticket }` gives the account
- `/login?google=...` means it did not work: show the message for `not_ready`,
  `cancelled`, `expired` or `failed` (Texts below)

### My account

Shows email, name, phone, language, how they sign in (password, Google or
both) and since when. Plus:

- **Change password** (has a password): current password and the new one,
  `POST /api/auth/password/change { old_password, new_password }` (logged in), or
  **Use a code instead** (code step, then set a password with the ticket)
- **Set a password** (Google only, no password yet): always the code step
  first, then set a password with the ticket. Never without the code
- **Devices logged in** (`GET /api/auth/sessions`): browser, last used, IP,
  "this device"; **Log out** on each other one (`POST /api/auth/sessions/end { id }`)
  and **Log out of all other devices** (`POST /api/auth/sessions/end-others`)
- **Log out** (`POST /api/auth/logout`)
- Sessions exist now: people stay logged in (cookie, sent by the browser on its
  own). On load, `GET /api/auth/me` says who is logged in (`401 not_logged_in`
  when nobody is). A logged-in person who opens `/` or `/login` goes to the
  app; one with no business yet goes to `/setup` (`FE-003`)

### Example responses

```json
POST /api/auth/start            {"next":"code","retry_after_seconds":60}
POST /api/auth/code/verify      {"ticket":"Gk3...","needs_password":true,"account":{...}}
POST /api/auth/password/login   {"account":{"email":"owner@example.com","name":null,"phone":null,"language":"ar","has_password":true,"has_google":false,"created_at":"2026-10-21T09:00:00.000Z"}}
Any error                       {"error":{"code":"wrong_code","field":"code"}}
Locked                          {"error":{"code":"locked","retry_after_seconds":840}}
```

### Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Headline (suggestion) | Run your shop by talking to it |
| Under it (suggestion) | One AI for your chats, orders, stock and cash. Made for coffee shops and small markets. |
| Google button | Continue with Google |
| Divider | or |
| Email button | Continue with email |
| Consent line | By continuing, you agree to the Terms of Service and Privacy Policy. |
| Email label | Email |
| Code title | Check your email |
| Code line | We sent a 6-digit code to {email}. It works for 10 minutes. |
| Resend | Send a new code / Send a new code in {seconds}s |
| Spam hint | Can't find it? Check your spam folder. |
| Set password title | Set a password |
| Password helper | At least 8 characters. |
| Log in title | Welcome back |
| Code instead | Email me a code instead |
| Another email | Use another email |
| Google account line | {email} signs in with Google. |
| Google not ready | Google sign-in is not ready yet. Use your email for now. |
| Google cancelled | Google sign-in was cancelled. |
| Google expired | Google sign-in took too long. Please try again. |
| Google failed | We couldn't sign you in with Google. Please try again. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `invalid_input` | A field is missing or wrong (`field` names it) | Please check what you typed. |
| `wrong_code` | The code does not match | That code isn't right. Check it and try again. |
| `code_expired` | Code older than 10 minutes, or used | This code has expired. Ask for a new one. |
| `too_many_tries` | 5 wrong codes | Too many wrong codes. Ask for a new one. |
| `too_soon` | New code within 60 seconds (`retry_after_seconds`) | Please wait a moment before asking for a new code. |
| `wrong_password` | Email and password do not match | The email and password don't match. |
| `locked` | 5 wrong passwords (`retry_after_seconds`) | Too many wrong passwords. Try again in a few minutes. |
| `no_password` | The account has no password yet | This account has no password yet. Use a code or Google. |
| `ticket_invalid` | The step took more than 10 minutes, or was done | This step has expired. Please start again. |
| `outside_failed` | Sending the email failed | We couldn't send the email. Please try again. |
| `limit_reached` | Too many requests (also: more than 10 email checks an hour from one place) | Too many tries. Please wait a little. |
| `not_logged_in` | No session, or it ended | You were logged out. Please log in again. |
| `bad_origin` | The request came from another site | This request was blocked. Please reload the page. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] The landing page has the top bar, the hero with the sign-up box and the product preview, the sections and the footer
- [ ] `/login` has the same box on its own page
- [ ] Signing up by email takes two steps after the email: the code, then the password
- [ ] Log in with a password, with "Email me a code instead"
- [ ] Google sign-in works there and back, including the error messages
- [ ] My account shows the info, with change and set password as described
- [ ] Every error code above shows its message
- [ ] Works on phone, tablet and laptop, in Arabic (right to left) and English
- [ ] Built in `app-web` (landing in `landing/`, the steps in `staff/setup/`), replacing the placeholders

## Open questions

- Who writes the Arabic texts (same open point as ALL-001)?
- Product name and logo are not chosen yet; use "PUA Graduation Project" until then.

## Changes

- 2026-10-21: Created.
- 2026-10-21: Backend changes since it was sent: sessions exist (stay logged in, devices, log out on My account); password change needs no email; errors carry only `code` and `field` (no `message`); the account has no `id`; Google errors return to `/login?google=...`; new errors `not_logged_in`, `bad_origin`; after sign-up a person with no business goes to `/setup` (`FE-003`). (backend)
