---
id: FE-007
title: Orders (the board of open orders by stage)
from: backend
to: frontend
cc: [agent]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Orders (the board of open orders by stage)

## Summary

The staff see every open order on one board, one column per stage, and move
each one along: accept, preparing, ready, collected or delivered. Orders come
from the counter, the website chat, Telegram and the phone; most chat orders
are taken by the AI.

## Context

- Design: `technical-structure.md` 17.2 (window 4), 8.2 (order life), 18.1
  (a customer orders on chat), 7.2 (stages come from the template), 7.3
  (packs `orders`, `delivery`, `tables`, `options`).
- The frame (full page, the AI panel, previews): `FE-005`.
- Card: `order` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/orders`, one order at `/orders/{number}`. Shows only when the pack
  `orders` is on
- Owner, manager and staff use it; a viewer sees it read-only

### 1. The board

- **One column per stage.** The stages come from the backend for each
  business (the template), never written in the app. A coffee shop: New,
  Preparing, Ready, then Collected or Delivered. A small market: New, Packed,
  Out for delivery, Delivered
- **Each order shows:** the number (#1042), the customer's name, where it came
  from (counter, website, Telegram, phone), the type (pickup, delivery, dine
  in with the table), the lines with their options, the total, paid or not,
  how long ago it came, and the ready time promised. **Taken by the AI** is
  marked
- **Next stage** with one tap (or drag it to the next column). A move the
  backend refuses (`conflict`) puts it back with the reason
- **New orders arrive by themselves**, without reloading, with a sound the
  person can turn off. The sidebar count updates (`Orders 4`)
- **Filters:** today, where it came from, type
- **Done today:** a list of today's collected, delivered and cancelled
  orders under the board

### 2. One order

Opens from the board (as a page on phones):

- Everything above, plus the delivery address and fee (pack `delivery`), the
  note, the payment, and the history of moves (who moved it and when, the AI
  or a person)
- **Accept** (new orders), **Next stage**, **Edit** (add or remove lines, only
  before preparing), **Cancel** (with a reason; a paid order needs approval),
  **Discount** (above the limit needs approval), **Record payment** (cash,
  card, wallet)
- **Open customer** (`/customers/{id}`) and **Open chat** (`/inbox/{id}`) when
  it came from a chat

### 3. A new order by phone

**New order** opens a short form: customer (find by phone or name, or new),
items with sizes and options, type, address for delivery, note. It uses the
same item picker as the sales screen (`FE-006`).

### 4. The AI on this page

The AI panel (`FE-005`) knows the selected order. "Mark #1042 ready", "how
many orders are late?", "cancel it, the customer called". A change it proposes
shows on the board as waiting until approved.

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Step | System call |
|------|-------------|
| The board, filters, done today | `erp.list_orders` |
| One order | `erp.get_order` |
| Next stage, accept | `erp.update_order_status` |
| Edit | `erp.edit_order` |
| Cancel | `erp.cancel_order` |
| Discount | `erp.apply_discount` |
| Payment | `erp.record_payment` |
| New order by phone | `crm.find_customer`, `crm.create_customer`, `erp.search_items`, `erp.create_order` (with a retry key) |
| New orders arriving | Events `OrderCreated`, `OrderReady`; how the app hears them comes in the follow-up |

### Example responses (draft: the real routes come in a follow-up)

```json
erp.list_orders {"status": "open"}
{
  "stages": [
    {"key": "new", "label": "New"},
    {"key": "preparing", "label": "Preparing"},
    {"key": "ready", "label": "Ready"},
    {"key": "collected", "label": "Collected", "final": true}
  ],
  "orders": [
    {"number": 1042, "stage": "new", "customer": {"id": "cu_3", "name": "Mariam"},
     "channel": "telegram", "type": "pickup", "table": null,
     "lines": [{"name": "Latte, medium", "options": ["Oat"], "qty": 2, "line_total_cents": 17000},
               {"name": "Croissant", "options": [], "qty": 1, "line_total_cents": 4500}],
     "total_cents": 21500, "currency": "EGP", "paid": false,
     "created_at": "2026-10-23T09:12:00.000Z", "ready_at": "2026-10-23T09:30:00.000Z",
     "created_by": "ai", "conversation_id": "cv_9"}
  ]
}

erp.update_order_status {"number": 1042, "to": "preparing"}
{"order": {"number": 1042, "stage": "preparing"}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Page | Orders / New order / Done today |
| Filters | Today / Counter / Website / Telegram / Phone / Pickup / Delivery / Dine in |
| Order | Order #{number} / {minutes} min ago / Ready at {time} / Paid / Not paid / Table {n} / Taken by the AI |
| Actions | Accept / Next stage / Edit / Cancel / Discount / Record payment / Open customer / Open chat |
| Cancel | Why cancel? / The customer cancelled / Out of stock / Other |
| History | {who} moved it to {stage} at {time} |
| Sound | Sound on / Sound off |
| Empty | No open orders. |
| Waiting | Waiting for approval |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `conflict` | The order moved already, or it is past the stage for editing | This order changed. Here is the latest. |
| `needs_approval` | Cancelling a paid order, a discount above the limit | Sent for approval. |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `pack_off` | `orders` is off | This part is off for your business. |
| `not_found` | No order with this number | We couldn't find this order. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/orders` shows the board with the stages from the response, not written in the app
- [ ] An order moves to the next stage by tap or drag; a refused move goes back with the reason
- [ ] New orders show without reloading, with a sound that can be turned off
- [ ] `/orders/{number}` shows the order with accept, edit, cancel, discount and payment
- [ ] A new phone order can be made, sent once (retry key)
- [ ] Orders taken by the AI are marked
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- How the app hears new orders live (server-sent events or polling): backend,
  in the follow-up.

## Changes

- 2026-10-23: Created.
