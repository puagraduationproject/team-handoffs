---
id: FE-003
title: Business setup, one question per screen
from: backend
to: frontend
cc: [security]
status: open
priority: high
created: 2026-10-21
updated: 2026-10-21
author: backend
---

# Business setup, one question per screen

## Summary

Right after sign-up, the owner sets up their business. Today it is one plain
form at `/setup`; we need it as short steps, **one question per screen**, that
the owner goes through before they can use the app. The backend is built and
live on the test address.

## Context

- Design: `technical-structure.md` section 7.7 (business setup and names).
- Backend routes, inputs, outputs and errors: `app-backend/api/app/README.md`.
  Error codes: `app-backend/contracts/errors/README.md`.
- The sign-up and log-in pages before it: `FE-002`.
- The current placeholder: `app-web/staff/setup/SetupPage.tsx` (one form,
  same questions, same order). Replace it.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### The rules

- **It cannot be skipped.** A logged-in person with no business always lands
  on `/setup`, whatever page they open. There is no "Later". The only way out
  is **Log out**, which every step shows
- **One question per screen**, in this order. A **Back** button on every step
  after the first keeps what was typed
- A progress hint (step 2 of 5, dots, a bar: your choice)
- Nothing is sent to the backend until the last step; then one call creates
  everything
- The username (`username@handle`) is **not shown** during setup. It comes
  from the person's name (or the email when the name has no Latin letters) and
  is shown and changed later in Settings

### The steps

| # | Step | Shown when | What it asks |
|:-:|------|------------|--------------|
| 1 | Your name | Only when the account has no name (`account.name` is null, from `GET /api/auth/me`). Google accounts usually skip it | One field |
| 2 | Business name | Always | One field, up to 80 characters |
| 3 | Business handle | Always | One field, filled in from the business name (Latin letters only; an Arabic name gives an empty box to type in). Small letters, digits and dashes, 3 to 30. Check it while typing (wait about 400 ms after the last key): `GET /api/app/handles/{handle}` |
| 4 | What kind of business | Always | One choice from the list below. No default: they must pick |
| 5 | Who works in it | Always | **Just me** (picked by default) or **Me and a team** |
| ✓ | Done | After the call works | "{Business name} is ready", then **Continue** to the app |

Then `POST /api/app/businesses`:

```json
{
  "your_name": "Ahmed Nassem",
  "name": "Nova Café",
  "handle": "nova",
  "business_type": "coffee_shop",
  "team_size": "alone"
}
```

`your_name` only when step 1 was shown. When it answers `handle_taken`, go back
to step 3 and show the message there.

### The kinds of business (step 4)

| Value | Label |
|-------|-------|
| `coffee_shop` | Café or coffee shop |
| `restaurant` | Restaurant or food |
| `bakery` | Bakery or sweets |
| `small_market` | Grocery or small market |
| `pharmacy` | Pharmacy |
| `clothing` | Clothes and fashion |
| `electronics` | Phones and electronics |
| `beauty` | Salon, barber or beauty |
| `services` | Other services |
| `online` | Online shop |
| `other` | Something else |

For now a type is only a name: it does not change what the app shows yet.
Do not promise features next to each one (no "menu, tables, ..." hints).

Eleven choices is a lot for a phone screen; how to show them is yours (a grid,
icons, a list).

### Example responses

```json
GET  /api/auth/me                 {"account":{"email":"owner@example.com","name":null,"phone":null,"language":"ar","has_password":true,"has_google":false,"created_at":"2026-10-21T09:00:00.000Z"}}
GET  /api/app/businesses          {"businesses":[]}
GET  /api/app/handles/nova        {"handle":"nova","available":true,"reason":null}
GET  /api/app/handles/login       {"handle":"login","available":false,"reason":"reserved"}
GET  /api/app/handles/nova        {"handle":"nova","available":false,"reason":"taken"}
POST /api/app/businesses   (201)  {"business":{"name":"Nova Café","handle":"nova","business_type":"coffee_shop","team_size":"alone","role":"owner","username":"ahmed","display_name":"Ahmed Nassem"}}
Taken meanwhile           (409)   {"error":{"code":"handle_taken","field":"handle"}}
A field is wrong          (400)   {"error":{"code":"invalid_input","field":"your_name"}}
```

`reason` is `invalid`, `reserved`, `taken` or null. Every route needs the
person to be logged in (the session cookie is sent by the browser on its own);
without it they answer `401` `not_logged_in`: go to `/login`.

### Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Setup title | Set up your business |
| Under it | A few questions. You can change all of it later. |
| Step 1 | What's your name? |
| Step 2 | What's your business called? |
| Step 3 | Pick a handle for it |
| Step 3 helper | Its unique name in the app. Small letters, digits and dashes. |
| Handle free | Available |
| Handle `invalid` | 3 to 30 small letters, digits or dashes. |
| Handle `reserved` | This name is kept for the app. Pick another. |
| Handle `taken` | Another business has it. Pick another. |
| Step 4 | What kind of business is it? |
| Step 4 not picked | Pick what kind of business it is. |
| Step 5 | Who works in it? |
| Step 5 choices | Just me / Me and a team |
| Next / Back | Next / Back |
| Last step button | Build my business |
| Log out | Log out |
| Done title | {Business name} is ready |
| Done, with a team | Inviting your team comes in a later step. You'll find it in Settings. |
| Done button | Continue |

The kinds of business are in the table above.

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_logged_in` | No session, or it ended | You were logged out. Please log in again. (then `/login`) |
| `invalid_input` | A field is missing or wrong (`field` names it) | Please check what you typed. (on the step of that field) |
| `handle_taken` | Another business took the handle meanwhile | Another business has this handle. Pick another. (on step 3) |
| `limit_reached` | Too many businesses (5) or requests | Too many tries. Please wait a little. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/setup` asks one question per screen, in the order above, with Back and a progress hint
- [ ] Step 1 shows only when the account has no name
- [ ] The handle is filled in from the business name and checked while typing, with the four messages
- [ ] All eleven kinds can be picked; none is picked at first
- [ ] One `POST /api/app/businesses` at the end; `handle_taken` takes them back to step 3
- [ ] A logged-in person with no business cannot reach any other page; Log out works from every step
- [ ] The username is not shown anywhere in setup
- [ ] Works on phone, tablet and laptop, in Arabic (right to left) and English
- [ ] Built in `app-web/staff/setup/`, replacing `SetupPage.tsx`

## Open questions

- Who writes the Arabic texts (same open point as ALL-001)?
- Icons for the kinds of business: yours to pick, if you want them.

## Changes

- 2026-10-21: Created.
