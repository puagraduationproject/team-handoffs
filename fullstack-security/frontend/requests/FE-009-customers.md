---
id: FE-009
title: Customers (the list, the customer page, support cases, bookings)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Customers (the list, the customer page, support cases, bookings)

## Summary

Everyone who bought or chatted, as one record per person, whichever channel
they came from: what they usually order, their points, their debts, notes,
and everything that happened with them. Beside it, the support cases
(complaints and requests) and the bookings.

## Context

- Design: `technical-structure.md` 17.2 (window 6), 2.5 (one customer on
  every channel), 9.1 to 9.5 (CRM), 18.2 (a customer complains), 7.3 (packs
  `loyalty`, `support`, `bookings`, `credit`).
- The frame (full page, the AI panel, previews): `FE-005`.
- Card: `customer` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/customers`, one customer at `/customers/{id}`, cases at
  `/customers/cases` and `/customers/cases/{number}`, bookings at
  `/customers/bookings`
- Owner and manager; a viewer read-only. Staff do not see this page (they
  see customers inside the Inbox and on orders)
- Customer phone numbers are personal data: show them in full only to the
  roles the backend allows; otherwise the backend sends them hidden
  (`+20 10•• ••• 345`)

### 1. The list

- Name, area, how they reach us (phone, Telegram, website chat), number of
  orders, total spent, last visit, tags
- **Search** by name or phone. **Filters:** regulars, quiet (have not come for
  a while), new, by area, by spend
- **Add customer**: name, phone, area

### 2. The customer page

- **Top:** name, area, their identities (phone, Telegram, ...), tags, and
  **Possible duplicate: {name}** when the backend suggests one (merging comes
  in term 2: show the note, the button is "soon")
- **Numbers:** orders, total spent, average order, last order, first seen
- **Usual order** ("Latte, medium, oat milk")
- **Points and tier** and the next reward (pack `loyalty`); **Add points**
  with a reason (above a limit needs approval)
- **Balance on account** and limit (pack `credit`); **Record payment**;
  changing the limit always needs approval
- **Notes**, **Tags**, **Follow-ups** ("Call Mariam Thursday", for a person
  or the AI)
- **Timeline:** everything that happened, newest first: orders, messages,
  payments, notes, points, cases, bookings; each links to its page
- **Open cases** and **next booking**
- **Open chat** (`/inbox/{id}`), **New order** (`FE-007`)

### 3. Support cases (pack `support`)

- **List:** number, customer, type (complaint, request, question), status
  (open, waiting, resolved), priority, assigned to, opened when. Filters by
  status and by staff member. **Opened by the AI** is marked
- **One case:** the customer, the linked order and chat, the history and
  notes. **Assign**, **Priority**, **Add note**, **Issue voucher** (above a
  limit needs approval), **Resolve** with the result: apology, replacement,
  voucher, or refund (a refund always needs approval)

### 4. Bookings (pack `bookings`, basic in term 1)

- **Today** and **this week:** time, name, party size, deposit, status, note
- **New booking:** customer, date and time, party size; it checks the free
  times first. **Change** and **Cancel**

### 5. The AI on this page

The AI panel (`FE-005`) knows the selected customer or case: "what does
Mariam usually order?", "send her a sorry voucher", "who hasn't come back this
month?". A voucher, points or a message to a customer shows as waiting until
approved.

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Part | System calls |
|------|--------------|
| List, search | `crm.list_customers`, `crm.find_customer`, `crm.create_customer` |
| Customer page | `crm.get_customer`, `crm.update_customer`, `crm.get_timeline`, `crm.suggest_merge`, `crm.add_note`, `crm.tag_customer`, `crm.create_follow_up` |
| Loyalty | `crm.get_loyalty`, `crm.add_points` |
| Account | `erp.get_credit_account`, `erp.record_debt_payment`, `erp.set_credit_limit` (always approval) |
| Cases | `crm.list_cases`, `crm.get_case`, `crm.update_case`, `crm.add_case_note`, `crm.issue_voucher`, `crm.resolve_case` |
| Bookings | `crm.list_bookings`, `crm.check_availability`, `crm.create_booking`, `crm.update_booking`, `crm.cancel_booking` |

### Example responses (draft: the real routes come in a follow-up)

```json
crm.list_customers {"filter": "regulars"}
{"customers": [{"id": "cu_3", "name": "Mariam", "area": "Maadi", "channels": ["telegram", "phone"],
  "orders_count": 20, "total_spent_cents": 182000, "currency": "EGP", "last_order_at": "2026-10-22T09:12:00.000Z", "tags": ["regular"]}]}

crm.get_customer {"id": "cu_3"}
{"customer": {"id": "cu_3", "name": "Mariam", "area": "Maadi",
  "identities": [{"type": "phone", "value": "+20 10•• ••• 345"}, {"type": "telegram", "value": "@mariam"}],
  "stats": {"orders_count": 20, "total_spent_cents": 182000, "avg_order_cents": 9100, "first_seen_at": "2026-06-02T10:00:00.000Z"},
  "usual_order": "Latte, medium, oat milk",
  "loyalty": {"points": 140, "tier": "Silver", "next_reward": {"name": "Free drink", "points": 200}},
  "credit": null,
  "tags": ["regular"], "notes": [{"text": "Allergic to nuts", "by": "Ahmed", "at": "2026-09-01T12:00:00.000Z"}],
  "open_cases": [{"number": 12, "type": "complaint", "status": "open"}],
  "possible_duplicate": null}}

crm.get_timeline {"customer_id": "cu_3"}
{"activities": [{"type": "order", "summary": "Order #1042, 215 EGP", "ref": {"type": "order", "number": 1042}, "actor_type": "ai", "occurred_at": "2026-10-23T09:12:00.000Z"}]}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Page | Customers / Support cases / Bookings / Add customer / Search by name or phone |
| Filters | Regulars / Quiet / New / Area / Spend |
| Customer | Orders / Total spent / Average order / Last order / First seen / Usual order / Possible duplicate: {name} / Merge (soon) |
| Loyalty | {points} points / {tier} / {n} points to {reward} / Add points / Reason |
| Account | Balance / Limit / Record payment / Change limit |
| Engagement | Notes / Add note / Tags / Follow-ups / New follow-up / Timeline |
| Cases | Complaint / Request / Question / Open / Waiting / Resolved / Assign / Priority / Issue voucher / Resolve / Apology / Replacement / Voucher / Refund / Opened by the AI |
| Bookings | Today / This week / New booking / Party size / Deposit / Change / Cancel / No free time then. |
| Actions | Open chat / New order |
| Waiting | Waiting for approval |
| Empty | No customers yet. They appear when someone orders or chats. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `needs_approval` | Points, voucher above the limit, a refund, a credit limit | Sent for approval. |
| `limit_reached` | Above the person's limit | This is above your limit. It needs the owner. |
| `invalid_input` | A wrong phone or a missing field | Check {field}. |
| `conflict` | The booking time was taken, or the case changed | This changed. Here is the latest. |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `pack_off` | The part's pack is off | This part is off for your business. |
| `not_found` | The customer or case is gone | We couldn't find this. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] The list searches and filters; a customer can be added
- [ ] The customer page shows numbers, usual order, points, account, notes, tags, follow-ups and the timeline
- [ ] Cases can be assigned, noted and resolved; refunds and big vouchers wait for approval
- [ ] Bookings for today and the week; a new booking checks free times
- [ ] Each part hides when its pack is off; phone numbers show only as the backend sends them
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- Which roles see full phone numbers: Security decides.

## Changes

- 2026-10-23: Created.
