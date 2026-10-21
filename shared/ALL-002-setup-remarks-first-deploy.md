---
id: ALL-002
title: Business setup, remarks on the first deploy
from: everyone
to: everyone
cc: []
status: in-progress
priority: high
created: 2026-10-21
updated: 2026-10-21
author: moe
---

# Business setup, remarks on the first deploy

## Summary

Remarks from trying the business setup (`/setup`) on the first deploy: what to
change in the kinds of business and the handle, five bugs, what to remove, what
to add, and the setup we aim for. Each item names the team that owns it.
Several items are decisions for the whole team; they are under Open questions.

## Context

- Design: `technical-structure.md` sections 7.7 (business setup and names),
  7.2 and 7.3 (templates and packs), 2.1 (retry keys), 17.1 (the first Home),
  20 (where it runs: the shop page address)
- The setup request to frontend: `fullstack-security/frontend/requests/FE-003-business-setup-steps.md`.
  Several items below change it; backend updates FE-003 once they are agreed
- The Home request: `fullstack-security/frontend/requests/FE-004-home-chat-shell.md`
- Code: `app-web/staff/setup/SetupPage.tsx`, `app-web/staff/App.tsx`,
  `app-backend/kernel/index/businesses/service.ts`, `app-backend/gateway/doors/app/setup.ts`
  (they were `modules/core/businesses/service.ts` and `api/app/setup.ts` when
  this was written)
- Tried with: your name `moe`, business `tree`, handle `tree-smouha`, kind
  Pharmacy, Just me

Owners: **BE** backend, **FE** frontend, **SEC** security, **Design** the
technical structure, **Market** the market lead.

## Where it stands (2026-10-21)

Built in `app-backend` (`e197599` to `08e1263`) and `app-web` (`7176da2`,
`87e4621`), with tests (64 backend tests pass) and the whole setup tried in a
browser in both languages. Not deployed yet. Where a team decision was open, the
cheapest default was built and is marked **proposal**: change it here and the
code follows.

| # | Item | Status | What was done, or what is left |
|:-:|------|--------|--------------------------------|
| 1.1 | Grocery apart from small market | Done | `grocery` is its own kind |
| 1.2 | Start with a few kinds | Done, **proposal** | Six: café or coffee shop, small market, grocery, bakery or sweets, restaurant, something else. Market to confirm |
| 1.3 | Link kinds to templates | Done | `template_key` is set: café to `coffee_shop`, small market and grocery to `small_market` (**proposal**, open question 2), the rest null. Existing businesses were updated too. Packs are not built yet, so a template still turns nothing on; it is recorded and sent in `BusinessCreated` for when they are |
| 1.4 | Be honest about the rest | Done | Kinds with a template say "Ready for you"; picking another one shows "We're starting with cafés and markets..."; the done screen says "{name} is set up" instead of "is ready" |
| 2.1 | Say what the handle is for | Done | Under the business name, as they type: "Your shop page: `<domain>/kafeh-elnil`" and "Team log-ins look like: `moe@kafeh-elnil`". The second label differs from the text above because the example is the owner's own username |
| 2.2 | Arabic names must work | Done | In one place in the backend (`kernel/index/businesses/names.ts`), called by the app through `GET /api/app/names`: `كافيه النيل` gives `kafeh-elnil`. When a name gives less than 3 letters: `shop-` and four digits (not `cafe-`: the kind is asked after the name) |
| 2.3 | "You can change all of it later" | Done (text) | The promise is gone: "A few short questions." The handle says it goes into links and QR codes, "so pick it with care". Whether a handle can change is still open |
| 2.4 | Taking someone else's name | Done, list to grow | Reserved: our own words (35), 72 future paths (`menu`, `order`, `pay`, `checkout`, `invite`, `join`, `verify`, `official`, ...) and 58 brands (`starbucks`, `costa`, `cilantro`, `seoudi`, `carrefour`, `talabat`, `fawry`, ...). Report and reclaim: a draft in `app-backend/gateway/doors/app/README.md`. Our brand name goes in once chosen |
| 2.5 | Shop address namespace | Open (Design) | Not decided. The app writes the address in one function (`shopAddress` in `SetupPage.tsx`), so the decision changes one line there |
| 2.6 | Branches | Done (text), **proposal** | Under the handle: "One name for the whole business. Branches come later." |
| 3.1 | Lands on My account | Done | Setup ends on the done screen, then Home (`/home`). Home is a stand-in (`app-web/staff/home/HomePage.tsx`) until FE-004 replaces it; noted in FE-004. A logged-in owner opening `/`, `/login` or `/setup` goes to `/home` |
| 3.2 | A retry makes a second business | Done | `idempotency_key`: the app makes one per setup; a retry with it gets the same business back (`201`), even when two tries arrive at once. Tried in the browser with a lost reply: one business |
| 3.3 | Wrong message at 5 businesses | Done | `business_limit` (403): "One account can have up to 5 businesses." |
| 3.4 | Invited staff forced into setup | Waiting for invites | There is no invite yet to check (no `invite` table). The rule is written into FE-003; `invite` and `join` are reserved handles. Design to add the exception to 7.7 |
| 3.5 | Handle stops following the name | Done | Clearing the handle makes it follow the name again: at the next change of the name, or when the box loses focus |
| 4.1 | "Who works in it?" | Removed | `team_size` is optional and null for new businesses; old businesses keep their answer |
| 4.2 | Pharmacy | Removed | Old pharmacy businesses (`tree-smouha`) became "something else", with "Pharmacy" kept in `sells` |
| 4.3 | Online shop | Removed | The same. "Do you sell online?" is not asked yet |
| 4.4 | Two vague "other" choices | Merged | One "Something else"; "Other services" businesses became "something else" too |
| 4.5 | "{name} is ready" screen | Replaced | By the done screen (5.1) |
| 5.1 | The done screen as the reward | Done | The shop page address, the three first-step buttons (they open `/home?start=menu_photo`, `add_product`, `connect_telegram`), "Go to Home", and "Arabic · EGP · Cairo time" |
| 5.2 | Language before anything else | Done | Step 1, Arabic picked by default, and a switch at the top of every step. Saved on the account at once (`POST /api/auth/language`). Every setup text is in Arabic; **draft** texts, see Open questions |
| 5.3 | "Where is it?" | Done, **proposal** | City (Egypt's 27 governorates, Cairo, Giza and Alexandria first) and area (typed). Stored as the business's first location (`core.location`, built now) |
| 5.4 | "What do you sell?" | Done | After "Something else"; optional; kept in `sells` and sent in `BusinessCreated` |
| 5.5 | "What should the AI call you?" | Done | Replaces "Your name"; asked only when the account has no name |
| 5.6 | Funnel numbers | Done | The app reports each step once per account; the view `core.setup_funnel` gives, per step, how many accounts reached it and the median seconds from their first step |
| 6 | The setup we aim for | Done | Language, (what should the AI call you), business name with the handle, kind, where, done |

**What the design needs (for Design to write):** 7.7 (the new questions and
their order, no team question, the invite exception, the handle text, which
kinds link which template); 7.1 (`business.sells`, `business.idempotency_key`,
`location.city`, `setup_step`, `team_size` can be empty); 17.2 window 1; 20
(the shop page address, 2.5).

## Requirements

### 1. Kinds of business

1. **Separate grocery from small market** (BE, FE, Market). They are
   fundamentally different businesses and should be two choices, not one
2. **Start with a few kinds and grow the list** (BE, FE, Market). Begin with a
   small list we test on as we go, and add kinds later. **If you disagree,
   say so in this file**
3. **Link the kinds we support to their templates** (BE). Today
   `template_key` is always `null`, so picking "coffee shop" turns nothing
   on. This breaks "Pick coffee shop and it is ready" (1.5). The column and
   its check already exist; `coffee_shop` and `small_market` are term 1
   templates
4. **Be honest about the rest** (FE). A kind with no template yet should not
   look ready. For example: "We're starting with cafés and markets. You'll get
   the basics now, and we'll tell you when yours is ready"

### 2. The handle

1. **Say what it is for** (FE). "Its unique name in the app" means nothing to
   a café owner. The handle goes into the shop page address, invite links and
   the team's log-in (`ahmed@nova`). Show it as they type:

   ```
   Your shop page: <domain>/tree-smouha
   Your team logs in as: moe@tree-smouha
   ```

   That one preview explains the field and leads people to pick a good handle
2. **Arabic names must work** (FE, BE). `handleFrom` removes everything that
   is not Latin, so "كافيه النيل" gives an empty box. A large share of our
   users will type the name in Arabic. Transliterate it (كافيه النيل →
   `kafeh-elnil`), with a fallback when that gives too little (`cafe-4821`).
   If the backend should do this too (for the AI or imports), it lives in one
   place and the app calls it
3. **"You can change all of it later" is not true for the handle** (FE,
   Design). Once a café prints a QR code or shares its Telegram link,
   changing the handle breaks all of it. Either we make it true (old handles
   keep redirecting; QR codes, invite links and the Telegram link follow the
   new handle) or the text stops promising it. Decision under Open questions
4. **Taking someone else's name** (BE, SEC). The reserved list only covers
   system words. Anyone can register `cilantro`, `costa`, `seoudi`,
   `carrefour` or `starbucks`. Add:
   - a list of known Egyptian chains and brands, plus a report-and-reclaim
     process
   - our own brand name, once we choose it
   - paths we will need later: `menu`, `order`, `orders`, `pay`, `checkout`,
     `invite`, `join`, `careers`, `features`, `faq`, `contact`, `legal`, `sw`,
     `manifest`, `oauth`, `telegram`, `whatsapp`, `verify`, `official`, ...
5. **The shop page address shares one namespace with our own site** (Design,
   BE). With `<domain>/<shop>`, every page the landing site adds later can
   clash with a business that already has that handle. `<shop>.<domain>`, or
   a prefix like `<domain>/s/<shop>`, removes the problem for good. Cheap to
   decide now, expensive later (section 20)
6. **Branches** (FE, Design). If we account for businesses with more than one
   branch, the handle step should say: "One name for the whole business.
   Branches come later." Otherwise owners will add the area to the handle
   (`tree-smouha`) and a chain will end up as two businesses, with its
   customers, loyalty and stock split

### 3. Bugs and behaviour problems

1. **After "Build my business", the owner lands on My Account, not Home**
   (FE). `App.tsx:59` and `App.tsx:78` send a new owner to their devices and
   password settings. They should land on the first-time Home (17.1:
   "Welcome, Ahmed. Nova Café is ready. Let's add what you sell first"). Home
   is not built yet; note this target in FE-004
2. **A retry can create a second business** (BE, FE). `POST
   /api/app/businesses` has no retry key. If the request succeeds but the
   reply is lost (common on mobile data), the retry gets `handle_taken` for
   the person's own business; they pick a new handle and end up with two
   businesses. This breaks rule 2.1. Fix: accept an `idempotency_key`, or on
   `handle_taken` check whether that handle is already theirs and treat it as
   success
3. **The 5-business limit shows the wrong message** (BE, FE).
   `service.ts:103` throws `limit_reached` with 429, which the app shows as
   "Too many tries. Please wait a little." Waiting will never help. Give it
   its own code, `business_limit` (see Error codes)
4. **Invited staff will be forced to create a business** (FE, BE, Design).
   The rule "no business, so `/setup`, and only Log out gets you out"
   (`App.tsx:56`) traps a barista who signs up from an invite link, and anyone
   removed from their only business. Invites are a term 1 feature, so the rule
   needs an exception now: an account with a pending invite goes to the
   invite, not to setup. Section 7.7 should say so too
5. **The handle stops following the name after one edit** (FE). If someone
   edits the handle and then clears it, `handleEdited` stays true, so the
   field never follows the business name again. Small, but annoying

### 4. What to remove

1. **"Who works in it?"** (FE, BE). Remove it for now, or give it a real
   effect. Today its only effect is one sentence pointing to Settings, where
   there is nothing yet. A yes or no "team" also loses what matters later (3
   staff or 15)
2. **Pharmacy** (BE, FE). In Egypt, drug prices are set by the government and
   prescriptions are regulated. A customer agent answering "can I take X with
   Y?" is a medical and legal risk. Remove it, or block it until there is a
   template with hard rules (no medical advice, hand off at once)
3. **"Online shop"** (BE, FE). It is a way of selling, not a kind of
   business: an online clothes shop is "clothes". Remove it, or make it a
   later question ("Do you sell online?")
4. **"Other services" and "Something else"** (BE, FE). Two vague choices next
   to each other confuse people. Merge them into one
5. **The "{name} is ready" screen as it is now** (FE). A click with nothing on
   it. Remove it, or make it the reward (5.1)

### 5. What to add

1. **The done screen as the reward** (FE). Their shop page address (this is
   when the handle starts to mean something), the three first-step buttons
   from 17.1 (send a photo of your menu, add a product, connect Telegram),
   and one line with the defaults: Arabic · EGP · Cairo time
2. **Language before anything else** (FE). The whole flow is in English,
   while the account defaults to `ar` and the design says Arabic first. A
   language switch at the top of setup, Arabic picked by default
3. **"Where is it?"** (BE, FE, Market): city and area, in place of "Who works
   in it?". One picker, useful from the first day: the customer agent can
   answer "where are you?", delivery areas start from it, the shop page needs
   it, and the Market team sees which areas sign up. Backend stores it on the
   business's first location (the `location` table in 7.1, not built yet)
4. **"What do you sell?"** (BE, FE, Market): a short text box after
   "Something else". Free market research that tells us which template to
   build next; with the term 1 targets (30 to 50 sign-ups), the mix of kinds
   is real evidence for the presentation
5. **"What should the AI call you?"** (FE, optional) in place of "Your name".
   It gives the field a purpose and sets the tone ("Ahmed" or "أستاذ أحمد").
   Asked only when the account has no name, as today
6. **Funnel numbers** (BE, FE). Record which step people stop at. Cheap, and
   it shows whether setup really takes minutes (1.5)

### 6. The setup we aim for

| # | Step | Notes |
|:-:|------|-------|
| 1 | Language | Arabic picked by default |
| 2 | Business name | The handle is filled in from it (transliterated), with the live preview (2.1); "Change" to edit it |
| 3 | Kind of business | A short list (1.2); the kinds with a template marked ready; "Something else" with "What do you sell?" |
| 4 | Where is it? | City and area |
| ✓ | Done | The shop page address, the three first-step buttons, then Home |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `business_limit` | New. The account already has 5 businesses (replaces `limit_reached` here) | One account can have up to 5 businesses. |
| `handle_taken` | Unchanged, but no longer sent when the handle belongs to the person's own business from a retried request (3.2) | Another business has this handle. Pick another. |

## Done when

- [x] Grocery and small market are two separate kinds
- [ ] The kinds list is agreed (1.2), and the kinds with a template link to it (`template_key` set). Linked: done; the list waits for Market
- [x] Kinds with no template are shown honestly
- [x] The handle step shows the shop page address and the team log-in as they type
- [x] An Arabic business name gives a usable handle
- [x] The text "You can change all of it later" matches what is true for the handle
- [x] The reserved list has the brands and future paths; a report-and-reclaim process is written down (a draft; its owner is open)
- [ ] The shop page address format is decided and written in section 20
- [x] A retried create never makes a second business
- [x] The 5-business limit answers `business_limit` with its own message
- [ ] An account with a pending invite goes to the invite, not to setup; section 7.7 says so
- [x] The handle follows the name again once the field is cleared
- [x] After setup, the owner lands on Home (noted in FE-004 until Home is built)
- [x] Pharmacy, Online shop and the duplicate "other" are removed or handled as above
- [x] "Who works in it?" is removed or has a real effect
- [x] The done screen shows the shop address, the first-step buttons and the defaults
- [x] Setup starts with the language, Arabic by default
- [x] FE-003 is updated to match everything agreed here

## Open questions

- **How many kinds, and which?** (1.2) Market's interviews should decide.
  Disagreement with starting small goes here. *Built for now:* the six in the
  table above. A kind is one line in the backend and one label in the app
- **Grocery and small market: two templates, or one template with different
  packs?** The design's way would be one `small_market` template with
  different packs on (for example credit on for a grocery, barcodes on for a
  market); a new template only if they need different stages, fields or skills.
  *Built for now:* the design's way, `grocery` starts from `small_market`
- **Can a handle change?** If yes: old handles redirect, and are never given
  to another business. If no: say so in setup. *For now* setup promises
  nothing and asks to pick it with care. Backend's suggestion: no change in
  term 1; later yes, with redirects and old handles never given away
- **The shop page address:** `<shop>.<domain>`, `<domain>/s/<shop>`, or keep
  `<domain>/<shop>` with a long reserved list? Both of the first two end the
  clash: `<shop>.<domain>` reads best on a QR code but needs wildcard DNS and
  certificates; `<domain>/s/<shop>` needs nothing new. The reserved list now
  covers the paths we know of, which is enough until this is decided
- **Who keeps the brand list and handles reclaim requests?** (SEC or BE)
  The draft process is in `app-backend/gateway/doors/app/README.md`
- **Branches:** do we say "branches come later" in setup now, or wait for the
  `branches` pack (term 2)? *Built for now:* it says so; remove the line if
  the team prefers to wait
- **Where the city and area list comes from**, and how detailed it is.
  *Built for now:* the 27 governorates and a typed area. A finer list can
  replace it without changing what is stored
- **The Arabic texts** for all the steps: same open point as ALL-001.
  Backend wrote a first draft in `app-web/shared/i18n/ar.ts`; it needs a
  native review

## Changes

- 2026-10-21: Created from the remarks on the first deploy.
- 2026-10-21: Built by backend (with Claude) in `app-backend` and `app-web`;
  status `in-progress`. Added "Where it stands". Still open: the kinds list
  (Market), the shop page address (Design), invites (3.4), handle changes, the
  brand list's owner, the Arabic texts. FE-003 and FE-004 updated to match.
