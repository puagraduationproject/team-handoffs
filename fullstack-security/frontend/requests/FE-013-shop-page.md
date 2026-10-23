---
id: FE-013
title: The shop page (the menu and the chat on one page, for customers)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# The shop page (the menu and the chat on one page, for customers)

## Summary

Each business gets a public page its customers open from a link or a QR
code: the menu with live prices, the opening hours and the location, and a
chat with the business's AI, which is the main way to order. No account
needed.

## Context

- Design: `technical-structure.md` 17.2 (window 13), 11.1 (store website),
  6.1 (the customer agent), 18.1 (a customer orders on chat), 7.7 (the
  handle), 22 (shop pages are built with Astro), 26 question 10 (the address).
- Cards the customer sees in the chat: `order`, `item`, `question` in
  `app-backend/contracts/cards/`.
- This page is for **customers**, not staff: no sidebar, no AI panel, no
  approvals.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`). Phones first: most
customers open it from a QR code or Telegram.

## Requirements

### Where

- `/{handle}` on the test address for now (`/nova-cafe`). The final address
  is still open (design 26, question 10); keep the handle in one place so the
  address can change
- Public, no log-in. Fast on a weak phone connection

### 1. The page

- **Top:** the business name, its kind and area, **open now** or **closed,
  opens at {time}**, and the banner when the owner set one ("Closed Friday
  until 2")
- **The menu:** by category, each item with name, photo, price (or "from 70"
  with sizes), and **Sold out** when it is. Only the items and categories the
  owner shows (pack `website`). Prices are always live
- **Info:** opening hours, address and a map link, phone, how to order
  (pickup, delivery with the areas and fee, dine in)
- **Order through the chat** button, always in reach

### 2. The chat

- Opens over the menu on phones, beside it on wide screens
- The customer types (Arabic, English or Franco) or taps an item's **Order**
  button, which starts the chat with that item
- The AI answers with text and cards: an **item** card (sizes and options to
  tap), an **order** card (lines, total, pickup or delivery, ready time) with
  **Confirm**, **Edit** and **Cancel**, and **question** cards with choices
- To confirm an order, the chat asks the customer's name and phone once
  (kept on this phone for next time)
- After **Confirm**: the order number and a link to its status page
  (`FE-014`)
- When the AI passes the chat to a person: "A team member will answer you
  here." Staff replies show in the same chat
- The chat stays when the customer reloads or comes back on the same phone
- Until the AI is connected: "The AI is not connected yet."

### System calls and routes (public door, draft)

The page talks to the **public door** (`api/public`), not the app door. The
customer agent behind it has only the customer system calls (design 8.5,
"Customer" column).

| Step | What |
|------|------|
| The page | `store.get_site`, `core.get_business`, `erp.list_categories`, `erp.search_items` (without cost), through one public route in the follow-up |
| The chat | Send a message and stream the answer: format with the AI agent team, in the follow-up |
| Confirm | `erp.create_order` (the customer's own, with a retry key), by the customer's tap on the card |

### Example responses (draft: the real routes come in a follow-up)

```json
GET (public route in the follow-up) /nova-cafe
{"business": {"name": "Nova Café", "kind": "Coffee shop", "area": "Maadi, Cairo", "phone": "+20 10 0000 0000",
  "open_now": true, "closes_at": "23:00", "banner": null,
  "hours": [{"day": "sat", "open": "08:00", "close": "23:00"}],
  "order_types": ["pickup", "delivery"], "delivery_areas": [{"name": "Maadi", "fee_cents": 2000}]},
 "categories": [{"id": "c_1", "name": "Hot drinks", "items": [
   {"id": "it_7", "name": "Latte", "photo_url": null, "from_price_cents": 7000, "available": true}]}],
 "currency": "EGP"}

order card in the chat
{"type": "order", "status": "draft", "lines": [{"name": "Latte, medium", "options": ["Oat"], "qty": 2, "line_total_cents": 17000}],
 "type_of_order": "pickup", "total_cents": 17000, "currency": "EGP", "ready_in_minutes": 15,
 "actions": ["confirm", "edit", "cancel"]}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Top | Open now / Closed, opens at {time} |
| Menu | from {price} / Sold out / Order |
| Info | Opening hours / Address / Open in maps / Call / Pickup / Delivery / Dine in / Delivery fee {amount} |
| Chat | Order through the chat / Ask or order here... / Send |
| Details | Your name / Your phone / We use it only for your orders. |
| Order card | Confirm / Edit / Cancel / Ready in about {minutes} minutes |
| Done | Your order #{number} is in. / Follow your order |
| Handoff | A team member will answer you here. |
| AI not connected | The AI is not connected yet. |
| Not found | This shop doesn't exist. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_found` | No business with this handle | This shop doesn't exist. |
| `conflict` | An item sold out before Confirm | {item} just sold out. |
| `invalid_input` | A wrong phone number | Check your phone number. |
| `limit_reached` | Too many messages in a short time | Please wait a moment before sending more. |
| — | Network down | We couldn't reach the shop. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/{handle}` shows the business, live menu, hours and info against the example response
- [ ] The chat opens from the button or an item's Order, and shows text, item, order and question cards
- [ ] Confirm sends the order once (retry key) and gives the number and the status link
- [ ] Name and phone are asked once and kept on the phone
- [ ] Phone first, fast on a weak connection; Arabic (right to left) and English

## Open questions

- The final shop page address (design 26, question 10).
- The message and streaming format for the public chat: with the AI agent
  team.
- How long the chat is kept on the customer's phone: Security decides.

## Changes

- 2026-10-23: Created.
