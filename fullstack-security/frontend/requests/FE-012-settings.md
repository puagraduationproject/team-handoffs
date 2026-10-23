---
id: FE-012
title: Settings (business, team, packs, channels, the AI's rules, knowledge, skills)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: medium
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Settings (business, team, packs, channels, the AI's rules, knowledge, skills)

## Summary

The owner's control room: the business profile and hours, the team and their
roles, which packs are on, the channels, what the AI may do alone, the
knowledge the AI answers from, and the business's skills. Owner only.

## Context

- Design: `technical-structure.md` 17.2 (window 9), 7.1 and 7.5 (Core), 7.3
  (the library of packs, turning on and off), 6.2 (the model picker), 6.4
  (knowledge), 6.6 (Not allowed, Ask first, Act alone, the defaults table),
  5 (skills), 10 (Channels).
- The frame (full page, the AI panel, previews): `FE-005`.
- Values: `app-backend/contracts/values/` (roles, AI permissions, channels).

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/settings`, one part per address: `/settings/business`,
  `/settings/team`, `/settings/packs`, `/settings/channels`, `/settings/ai`,
  `/settings/knowledge`, `/settings/skills`, `/settings/activity`
- **Owner only.** Others do not see it. My account (`/account`) stays
  separate: it is the person's, not the business's

### 1. Business

Name, handle (shown, not changed in term 1), kind, what it sells, location
(city and area), address, **opening hours** per day, delivery areas and fees
(pack `delivery`), language, currency and time zone (shown).

### 2. Team

- Members: name, `username@handle`, role (owner, manager, staff, viewer),
  joined when
- **Invite** by phone or email with a role; pending invites with **Cancel**
- **Change role** and **Remove**, each with one confirm line
- What each role can do (read-only table from design 17.2)

### 3. Packs (the library)

- Two lists: **On** and **In the library**, each pack with its name, one line
  of what it adds, and the term it comes in ("Term 2" packs show as soon)
- **Turn on:** shows the packs it needs, which turn on with it ("Recipes needs
  Stock. Turn on both?"), then starts its setup in the AI panel
- **Turn off:** says the data is kept and comes back. Refused while another
  pack that is on needs it, or while there is open work (an open shift, open
  orders, purchase orders not received): show the reason
- The template's packs are marked "Recommended for coffee shops"

### 4. Channels

- **Website chat:** on, with the shop page address and a QR code to print
- **Telegram** (pack `telegram`): **Connect** walks the owner through making a
  bot and pasting its key; then connected, the bot's name, **Disconnect**
- **WhatsApp, Instagram, Facebook:** "soon"
- **Auto replies** per channel: away, opening hours, greeting; each on or off
  with its text

### 5. The AI

- **What the AI may do alone:** one row per area (from design 6.6: answering
  customers, accepting chat orders, support and vouchers, waste and sales,
  drafts, sending purchase orders, prices and menu, public posts, a message to
  one customer, messages to many, money, people and setup), each with **Not
  allowed / Ask first / Act alone**. Rows marked "always ask" offer only Ask
  first or Not allowed
- **Limits:** the most the AI and managers may give or spend without asking
  (discount, voucher, purchase order, cash out, stock count)
- **Model:** per face (owner partner, customer agent, jobs): **Auto (our
  model)**; the others "soon". Each choice says where it runs and whether data
  leaves our server (design 6.2)
- **Tone:** how the AI talks to customers (friendly, formal), in a few words

### 6. Knowledge

FAQs and policies the customer agent answers from (refunds, allergies,
delivery areas, parking): a list of question and answer, or a short page each.
Add, edit, remove. This is the only text the AI searches; prices and stock
always come from the system (design 6.4).

### 7. Skills

- The business's skills: name, when it is used, on or off. Opening one shows
  its steps in plain words
- Changing a skill is done **by talking** ("before 8 am, check the milk
  first"): **Change by talking** opens the AI panel on that skill; the new
  version shows as waiting until the owner approves (design 5.3)

### 8. Activity

Who changed what, newest first: the person or the AI, the action, the thing,
before and after, when. Filter by person, the AI, area. Read-only.

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Part | System calls |
|------|--------------|
| Business | `core.get_business`, `core.update_business`, `core.list_locations` |
| Team | `core.list_members`, `core.invite_member`, `core.change_role`, `core.remove_member` |
| Packs | `core.list_packs`, `core.turn_on_pack`, `core.turn_off_pack` |
| Channels | `channels.list_channels`, `channels.set_auto_reply`; connecting Telegram comes in the follow-up |
| The AI | `ai.get_permissions`, `ai.set_permission`, `ai.list_models`, `ai.set_model` |
| Knowledge | Not in the list yet; the names come in the follow-up (`ai.search_knowledge` is the AI's) |
| Skills | `ai.load_skill`, `ai.save_skill`; listing comes in the follow-up |
| Activity | `core.get_audit_log` |

### Example responses (draft: the real routes come in a follow-up)

```json
core.list_packs
{"packs": [
  {"key": "stock", "name": "Stock", "adds": "The stock ledger, levels, waste, counts, forecasts", "status": "on", "needs": [], "term": 1, "recommended": true},
  {"key": "recipes", "name": "Recipes", "adds": "Ingredients, recipes, cost and margin per item", "status": "off", "needs": ["stock"], "term": 1, "recommended": true},
  {"key": "team", "name": "Team", "adds": "Rota, attendance, tasks", "status": "off", "needs": [], "term": 2, "recommended": false}
]}

core.turn_off_pack {"key": "shifts_cash"}
{"error": {"code": "conflict", "message": "open shift", "details": {"reason": "open_work", "what": "shift"}}}

ai.get_permissions
{"areas": [
  {"key": "answer_customers", "label": "Answer customers: menu, prices, hours, order status", "setting": "act_alone", "allowed": ["not_allowed", "ask_first", "act_alone"]},
  {"key": "message_many", "label": "Messages to many customers at once", "setting": "ask_first", "allowed": ["not_allowed", "ask_first"]}
],
 "limits": [{"key": "voucher", "label": "Voucher", "value_cents": 5000, "currency": "EGP"}]}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Parts | Business / Team / Packs / Channels / The AI / Knowledge / Skills / Activity |
| Business | Name / Handle / Kind / What you sell / City / Area / Address / Opening hours / Closed / Delivery areas / Fee |
| Team | Invite / Phone or email / Role / Owner / Manager / Staff / Viewer / Pending / Change role / Remove / Remove {name} from {business}? |
| Packs | On / In the library / Turn on / Turn off / Soon / Recommended for {kind} / {pack} needs {other}. Turn on both? / Your data is kept and comes back when you turn it on again. |
| Pack refused | {pack} can't be turned off while {what} is open. |
| Channels | Website chat / Your shop page / Print QR code / Connect / Connected / Disconnect / Auto replies / Away / Opening hours / Greeting |
| The AI | What the AI may do alone / Not allowed / Ask first / Act alone / Always asks / Limits / Model / Auto (our model) / Data stays on our server / Tone |
| Knowledge | Add a question / Question / Answer |
| Skills | When it is used / Change by talking / On / Off |
| Activity | The AI / {name} / Before / After |
| Saved | Saved. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `conflict` | A pack is needed by another, or has open work | {pack} can't be turned off while {what} is open. |
| `invalid_input` | A missing or wrong field | Check {field}. |
| `outside_failed` | Telegram refused the bot key | Telegram did not accept this key. Check it and try again. |
| `not_allowed` | Not the owner | You don't have access to this. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] Each part works against the example responses, owner only
- [ ] Packs turn on with what they need, and a refused turn-off shows why
- [ ] The AI's settings show one row per area with only the allowed choices
- [ ] Telegram can be connected and disconnected (with the follow-up routes)
- [ ] Changing a skill opens the AI panel; the new version waits for approval
- [ ] Activity lists who changed what, the AI included
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- The values of the limits (`app-backend/contracts/limits/`): the team and
  Security.
- The routes for connecting Telegram, knowledge and listing skills: backend,
  with the AI agent team.

## Changes

- 2026-10-23: Created.
