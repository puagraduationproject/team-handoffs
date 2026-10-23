---
id: FE-014
title: Order status (the customer follows the order from a link)
from: backend
to: frontend
cc: [security]
status: open
priority: medium
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Order status (the customer follows the order from a link)

## Summary

After ordering on the shop page, Telegram or the website chat, the customer
gets a link. It opens a small page with the order's stage and ready time,
which updates by itself.

## Context

- Design: `technical-structure.md` 17.2 (window 14), 11.1 (an order status
  page, sent as a link), 8.2 (order life; the stages come from the template).
- Linked from the shop page chat (`FE-013`) and from messages the business
  sends.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`). Phones first.

## Requirements

### Where

- `/{handle}/o/{code}`, where `{code}` is a long random code from the backend,
  **not** the order number, so nobody can guess another person's order
- Public, no log-in

### The page

- The business name, and **Order #{number}**
- **The stages** as steps, the current one marked: they come from the
  backend (a coffee shop: Received, Preparing, Ready, Collected; delivery:
  ... Out for delivery, Delivered), never written in the app
- **Ready at about {time}**, or "Ready now" when it is
- The lines and the total, pickup or delivery, paid or pay on collection
- **Cancelled** with the reason when it is
- **Updates by itself** (every few seconds is fine) while it is open
- **Chat with us** goes back to the chat on the shop page (`FE-013`)

### Route (public door, draft)

Through the public door, a read of the customer's own order (`erp.get_order`,
"own only" in design 8.5).

### Example response (draft: the real route comes in a follow-up)

```json
GET (public route in the follow-up) /nova-cafe/o/q7Zt3mK9vB2xR8pL
{"business": {"name": "Nova Café", "handle": "nova-cafe"},
 "order": {"number": 1042, "stage": "preparing",
   "stages": [{"key": "new", "label": "Received"}, {"key": "preparing", "label": "Preparing"}, {"key": "ready", "label": "Ready"}, {"key": "collected", "label": "Collected"}],
   "ready_at": "2026-10-23T09:30:00.000Z", "type": "pickup",
   "lines": [{"name": "Latte, medium", "options": ["Oat"], "qty": 2}], "total_cents": 17000, "currency": "EGP",
   "paid": false, "cancelled_reason": null}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Top | Order #{number} |
| Time | Ready at about {time} / Ready now |
| Payment | Paid / Pay when you collect / Pay on delivery |
| Cancelled | This order was cancelled: {reason} |
| Chat | Chat with us |
| Not found | We couldn't find this order. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_found` | A wrong or old code | We couldn't find this order. |
| `limit_reached` | Too many requests | Please wait a moment. |
| — | Network down | We couldn't reach the shop. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/{handle}/o/{code}` shows the order, its stages from the response and the ready time
- [ ] It updates by itself while open
- [ ] A wrong code shows "We couldn't find this order."
- [ ] Phone first; Arabic (right to left) and English

## Open questions

- How long the link keeps working after the order is done: Security decides.

## Changes

- 2026-10-23: Created.
