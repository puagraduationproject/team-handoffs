---
id: FE-003
title: Business setup, one question per screen
from: backend
to: frontend
cc: [security]
status: in-progress
priority: high
created: 2026-10-21
updated: 2026-10-21
author: backend
---

# Business setup, one question per screen

## Summary

Right after sign-up, the owner sets up their business: short steps, **one
question per screen**, that the owner goes through before they can use the
app. Since `ALL-002` (remarks on the first deploy) the steps are: language,
business name with its handle, kind of business, where it is, and a done
screen that leads to Home. The backend is built, and a working version of the
screens is in `app-web/staff/setup/SetupPage.tsx`, made by backend so the
remarks could be tried. **Its look is a placeholder:** it is yours to restyle
or rebuild.

## Context

- Design: `technical-structure.md` section 7.7 (business setup and names);
  the changes in `shared/ALL-002-setup-remarks-first-deploy.md` are not in the
  design yet.
- Backend routes, inputs, outputs and errors: `app-backend/gateway/doors/app/README.md`
  and `app-backend/gateway/shell/README.md` (the language). Error codes:
  `app-backend/contracts/errors/README.md`.
- The sign-up and log-in pages before it: `FE-002`. Home after it: `FE-004`.
- Texts: `app-web/shared/i18n/en.ts` and `ar.ts` (`setup`, `kinds`, `cities`).

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### The rules

- **It cannot be skipped.** A logged-in person with no business always lands
  on `/setup`, whatever page they open. There is no "Later". The only way out
  is **Log out**, which every step shows
- **Except an invite** (once invites are built): an account with a pending
  invite goes to the invite, not to setup (ALL-002 3.4)
- **One question per screen**, in this order. A **Back** button on every step
  after the first keeps what was typed
- A progress hint (step 2 of 5, dots, a bar: your choice)
- **In the person's language** from the first screen: Arabic by default,
  right to left, with a switch at the top of every step
- Only the last step creates the business, with one call. Before it, the app
  only saves the language when it is picked and reports each step for the
  funnel (both below)
- **One retry key for the whole setup.** Make an `idempotency_key` (a uuid)
  when setup opens and send it with every try of the last call, so a lost
  reply never makes a second business
- The username itself is not asked. It shows only inside the team log-in
  preview (`moe@kafeh-elnil`) and is changed later in Settings

### The steps

| # | Step | Shown when | What it asks |
|:-:|------|------------|--------------|
| 1 | Language | Always | العربية (picked by default, from `account.language`) or English. Saved at once: `POST /api/auth/language` |
| 2 | What should the AI call you? | Only when the account has no name (`account.name` is null). Google accounts usually skip it | One field, up to 80 characters |
| 3 | Business name | Always | One field, up to 80 characters. Under it, filled in as they type (wait about 400 ms after the last key): the handle from `GET /api/app/names`, shown as **Your shop page: `<domain>/<handle>`** and **Team log-ins look like: `<username>@<handle>`**, whether it is free, and "One name for the whole business. Branches come later." **Change** opens the handle to edit it (check it with `GET /api/app/handles/{handle}`); clearing it makes it follow the name again |
| 4 | Kind of business | Always | One choice from `GET /api/app/setup` (`business_types`), in that order, no default. The ones with `ready: true` are marked ready; picking another shows the "starting with cafés and markets" line. "Something else" adds **What do you sell?** (optional, up to 120 characters) |
| 5 | Where is it? | Always | City (one of `cities` from `GET /api/app/setup`, names in the app's texts) and area (typed, up to 60 characters) |
| ✓ | Done | After the call works | The title ("{name} is ready" for a ready kind, "{name} is set up" otherwise), the shop page address, three buttons: **Send a photo of your menu**, **Add a product**, **Connect Telegram** (to `/home?start=menu_photo`, `add_product`, `connect_telegram`), **Go to Home** (`/home`), and the line "Arabic · EGP · Cairo time" |

Each time a step is shown: `POST /api/app/setup/steps` with `step`:
`language`, `your_name`, `business_name`, `business_type`, `location`, then
`done` on the done screen. Ignore its errors.

Then `POST /api/app/businesses`:

```json
{
  "your_name": "Moe",
  "name": "كافيه النيل",
  "handle": "kafeh-elnil",
  "business_type": "coffee_shop",
  "city": "alexandria",
  "area": "سموحة",
  "idempotency_key": "6f1c2a8e-3b8d-4f0e-9a43-6a1d2c0e5b7f"
}
```

`your_name` only when step 2 was shown; `sells` only with `other`. When it
answers `handle_taken`, go back to step 3 with the handle open and show the
message there; for `invalid_input`, go to the step of its `field`.

### The kinds of business (step 4)

| Value | Label | Ready |
|-------|-------|:-----:|
| `coffee_shop` | Café or coffee shop | yes |
| `small_market` | Small market | yes |
| `grocery` | Grocery | yes |
| `bakery` | Bakery or sweets | |
| `restaurant` | Restaurant | |
| `other` | Something else | |

Show them in the order `GET /api/app/setup` gives. The list is a proposal for
Market to confirm (ALL-002 1.2). Do not promise features next to each one.

### Example responses

```json
GET  /api/auth/me                  {"account":{"email":"moe@example.com","name":null,"phone":null,"language":"ar","has_password":true,"has_google":false,"created_at":"2026-10-21T09:00:00.000Z"}}
POST /api/auth/language            {"account":{...,"language":"en",...}}
GET  /api/app/businesses           {"businesses":[]}
GET  /api/app/setup                {"business_types":[{"key":"coffee_shop","ready":true},{"key":"small_market","ready":true},{"key":"grocery","ready":true},{"key":"bakery","ready":false},{"key":"restaurant","ready":false},{"key":"other","ready":false}],"cities":["cairo","giza","alexandria","aswan","..."]}
GET  /api/app/names?name=كافيه%20النيل&your_name=Moe
                                   {"handle":"kafeh-elnil","available":true,"reason":null,"username":"moe"}
GET  /api/app/handles/starbucks    {"handle":"starbucks","available":false,"reason":"reserved"}
GET  /api/app/handles/nova         {"handle":"nova","available":false,"reason":"taken"}
POST /api/app/setup/steps          {"recorded":true}
POST /api/app/businesses   (201)   {"business":{"name":"كافيه النيل","handle":"kafeh-elnil","business_type":"coffee_shop","sells":null,"team_size":null,"role":"owner","username":"moe","display_name":"Moe"}}
Same key again             (201)   the same body: the first try's business
Taken meanwhile            (409)   {"error":{"code":"handle_taken","field":"handle"}}
A field is wrong           (400)   {"error":{"code":"invalid_input","field":"area"}}
Five businesses already    (403)   {"error":{"code":"business_limit"}}
```

`reason` is `invalid`, `reserved`, `taken` or null. Every route needs the
person to be logged in (the session cookie is sent by the browser on its own);
without it they answer `401` `not_logged_in`: go to `/login`.

### Texts (English; the Arabic drafts are in `app-web/shared/i18n/ar.ts`)

| Where | Text |
|-------|------|
| Under the first title | A few short questions. |
| Progress | Step {n} of {total} |
| Step 1 | Which language do you want? |
| Step 2 | What should the AI call you? |
| Step 2 hint | For example "Ahmed" or "Mr. Ahmed". |
| Step 3 | What's your business called? |
| Previews | Your shop page / Team log-ins look like |
| Handle | Handle; Small letters, digits and dashes. It goes into your links and QR codes, so pick it with care. |
| Handle line | One name for the whole business. Branches come later. |
| Handle free | Available |
| Handle `invalid` | 3 to 30 small letters, digits or dashes. |
| Handle `reserved` | This name is kept. Pick another. |
| Handle `taken` | Another business has it. Pick another. |
| Change | Change |
| Step 4 | What kind of business is it? |
| Ready kind | Ready for you |
| Kind not ready | We're starting with cafés and markets. You'll get the basics now, and we'll tell you when yours is ready. |
| Something else | What do you sell? For example: clothes, phones, a salon. You can leave it empty. |
| Step 5 | Where is it? City (Pick a city), Area (For example: Smouha) |
| Missing answers | Type your name. / Type the name of your business. / Pick a handle. / Checking the handle… / Pick what kind of business it is. / Pick the city. / Type the area. |
| Next / Back / last button | Next / Back / Build my business |
| Log out | Log out |
| Done title | {Business name} is ready / {Business name} is set up |
| Done, ready kind | Let's add what you sell first. |
| Done buttons | Start with one of these: Send a photo of your menu / Add a product / Connect Telegram; Go to Home |
| Done defaults | Arabic · EGP · Cairo time |

The kinds are in the table above; the 27 cities are in `cities` in the texts.
"You can change all of it later" is gone on purpose: it is not true for the
handle (ALL-002 2.3).

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_logged_in` | No session, or it ended | You were logged out. Please log in again. (then `/login`) |
| `invalid_input` | A field is missing or wrong (`field` names it) | Please check what you typed. (on the step of that field) |
| `handle_taken` | Another business took the handle meanwhile; never for your own retried request | Another business has this handle. Pick another. (on step 3) |
| `business_limit` | The account already has 5 businesses | One account can have up to 5 businesses. |
| `limit_reached` | Too many requests in a short time | Too many tries. Please wait a little. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [x] `/setup` asks one question per screen, in the order above, with Back and a progress hint
- [x] It starts with the language, Arabic by default, and the whole flow follows it (right to left in Arabic)
- [x] Step 2 shows only when the account has no name
- [x] The handle is made from the business name (Arabic too) and checked while typing, with the shop page and team log-in previews and the four messages
- [x] The handle follows the name again once it is cleared
- [x] The six kinds can be picked, the ready ones marked; none is picked at first; "Something else" asks what they sell
- [x] "Where is it?" asks the city and the area
- [x] One `POST /api/app/businesses` at the end with one retry key; `handle_taken` takes them back to step 3
- [x] The done screen shows the shop page address, the three first steps and the defaults, then Home
- [x] A logged-in person with no business cannot reach any other page; Log out works from every step
- [ ] Restyled by frontend, following ALL-001 (the look today is a placeholder)
- [ ] The Arabic texts reviewed by their owner (ALL-001)
- [ ] Tried on phone, tablet and laptop (done on a phone-size screen only)
- [ ] An account with a pending invite goes to the invite (when invites are built)

## Open questions

- Who writes the Arabic texts (same open point as ALL-001)?
- Icons for the kinds of business: yours to pick, if you want them.
- The kinds list, the shop page address and the city list: open in ALL-002.

## Changes

- 2026-10-21: Created.
- 2026-10-21: Updated to match ALL-002 (backend): language first; "What
  should the AI call you?"; the handle under the business name with previews,
  made from Arabic names too; six kinds with `ready`; "What do you sell?";
  "Where is it?" in place of "Who works in it?"; the done screen with the
  first steps, then Home; one retry key; `business_limit`; step reports for
  the funnel. A working version was built in `app-web` (its look is still
  yours). Status `in-progress`.
