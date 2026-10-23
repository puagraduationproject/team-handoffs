---
id: FE-006
title: The sales screen (counter sales, shifts and the cash count)
from: backend
to: frontend
cc: [security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# The sales screen (counter sales, shifts and the cash count)

## Summary

The café's register: the cashier or barista opens a shift, taps items into a
cart, takes the payment, and closes the shift with a cash count. It must be
fast at rush hour and work with big buttons on a tablet. The AI is not in the
way: its panel is hidden at first.

## Context

- Design: `technical-structure.md` 17.2 (window 3), 18.5 (the cashier does not
  go through the AI), 8.1 to 8.3 (orders, payments, shifts and cash), 7.3
  (packs `sales_screen`, `shifts_cash`, `options`, `barcodes`, `tables`,
  `loyalty`, `credit`).
- The frame (full page, the AI panel, previews): `FE-005`.
- Cards: `order`, `cash` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`). Sizes there: tablet
first, sales buttons around 64px.

## Requirements

### Where and who

- `/sales`. Shows only when the pack `sales_screen` is on
- Owner, manager and staff can sell; a viewer sees it read-only
- The AI panel is **hidden at first** here (`FE-005`)

### 1. Open the shift (pack `shifts_cash`)

- With no open shift, the screen asks first: "Open the shift" with the cash
  in the drawer now (a number in pounds)
- The open shift shows at the top: who opened it, since when
- Without the pack `shifts_cash`, there is no shift: selling starts at once

### 2. Sell

- **Categories** along one side, **item buttons** in the middle (name, price,
  photo if there is one). A search box; with the pack `barcodes`, a barcode
  scan adds the item
- **Sold out** items show as sold out and cannot be tapped. A long press
  offers **Sold out today** / **Back in stock**
- Tapping an item with **sizes or options** (pack `options`: milk type, extra
  shot, sugar) opens a quick choice; the price updates with each option
- **The cart:** lines with quantity (+ / −), options, line total, remove;
  subtotal, discount, total
- **Customer, optional:** find by phone or name, to add loyalty points (pack
  `loyalty`) or sell on account (pack `credit`). Shows their points and "1
  reward ready" when there is one, with **Use reward**
- **Order type:** take away by default; **dine in** with a table number with
  the pack `tables`
- **Discount:** a percent or an amount. Above the person's limit it goes for
  approval (`needs_approval`) and the sale waits, or the cashier removes it
- **Note** on the order (optional)

### 3. Pay

- **Cash:** the amount given, and the change to hand back
- **Card** and **wallet**: just the method (no payment device in term 1)
- **On account** (pack `credit`, needs a customer): shows the balance and the
  limit; above the limit it is refused (`limit_reached`)
- **Done** shows the order number (#1042) big, then a new empty cart
- A sale is sent **once**: each sale carries a retry key, so tapping Done
  twice or a dropped connection never makes two sales. If the network drops,
  keep the cart and say so; selling without internet is term 2

### 4. Cash in and out

**Cash in / Cash out** of the drawer during the shift, with an amount and a
reason ("bought milk"). Above a limit it goes for approval.

### 5. Close the shift (pack `shifts_cash`)

1. **Close the shift**, then count the cash in the drawer and type it
2. Then the result (the `cash` card): expected from sales, counted, and the
   difference ("50 EGP short"), with a note box
3. The shift is closed; the next person opens a new one

The AI never opens or closes a shift; a person does (design 8.3).

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Step | System call |
|------|-------------|
| Categories, items, search | `erp.list_categories`, `erp.search_items`, `erp.get_item` |
| Sold out / back | `erp.set_availability` |
| The sale, paid in one step | `erp.record_counter_sale` (with a retry key) |
| Discount | `erp.apply_discount` |
| Customer, points, reward | `crm.find_customer`, `crm.get_loyalty`, `crm.redeem_reward` |
| On account | `erp.get_credit_account`, `erp.charge_on_account` |
| The open shift, expected cash | `erp.get_shift` |
| Cash in and out | `erp.record_cash_movement` |
| Open and close the shift | Not in the list yet; the names come in the follow-up |

### Example responses (draft: the real routes come in a follow-up)

```json
erp.get_shift
{"shift": {"id": "sh_12", "opened_by": "Moe", "opened_at": "2026-10-23T08:00:00.000Z", "opening_cash_cents": 50000, "currency": "EGP"}}

erp.list_categories
{"categories": [{"id": "c_1", "name": "Hot drinks"}, {"id": "c_2", "name": "Pastries"}]}

erp.search_items {"category_id": "c_1"}
{"items": [
  {"id": "it_7", "name": "Latte", "photo_url": null, "available": true,
   "sizes": [{"id": "v_1", "name": "Medium", "price_cents": 7000}, {"id": "v_2", "name": "Large", "price_cents": 8500}],
   "option_groups": [{"id": "g_1", "name": "Milk", "required": true, "choose": 1,
     "options": [{"id": "o_1", "name": "Full cream", "price_cents": 0}, {"id": "o_2", "name": "Oat", "price_cents": 1500}]}]}
]}

erp.record_counter_sale
{"order": {"number": 1042, "total_cents": 15500, "currency": "EGP", "paid": true, "method": "cash", "change_cents": 4500, "points_added": 15}}

close the shift (name in the follow-up)
{"cash": {"expected_cents": 432000, "counted_cents": 427000, "difference_cents": -5000, "currency": "EGP"}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| No shift | Open the shift / Cash in the drawer now |
| Open shift | Shift opened by {name} at {time} |
| Search | Search items |
| Sold out | Sold out / Sold out today / Back in stock |
| Cart | Cart / Subtotal / Discount / Total / Note / Clear |
| Customer | Add customer / Phone or name / {points} points / 1 reward ready / Use reward |
| Type | Take away / Dine in / Table |
| Pay | Pay / Cash / Card / Wallet / On account / Given / Change |
| Credit | Balance {amount} of {limit} |
| Done | Order #{number} / New sale |
| Cash | Cash in / Cash out / Amount / Reason |
| Close | Close the shift / Count the cash in the drawer / Expected / Counted / Difference / {amount} short / {amount} over / Matches |
| Waiting | Waiting for approval |
| Offline | No connection. The cart is kept; try again when you are back online. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `pack_off` | `sales_screen` is off | This part is off for your business. |
| `needs_approval` | Discount or cash out above the limit | Sent for approval. |
| `limit_reached` | Above the customer's credit limit | This is above {name}'s credit limit. |
| `conflict` | An item became sold out, or the shift was closed elsewhere | {item} just sold out. / This shift was closed. |
| `invalid_input` | A wrong amount | Check the amount. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | No connection. The cart is kept; try again when you are back online. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/sales` opens a shift, sells, takes cash, card, wallet and on account, and closes the shift with a count, against the example responses
- [ ] Sizes and options change the price; sold-out items cannot be sold
- [ ] Tapping Done twice or losing the network never makes two sales (retry key)
- [ ] The cart survives a dropped connection
- [ ] Each pack's parts hide when the pack is off (`shifts_cash`, `options`, `barcodes`, `tables`, `loyalty`, `credit`)
- [ ] Tablet first, big buttons; works on a phone; Arabic (right to left) and English
- [ ] The AI panel is hidden at first and can be opened (`FE-005`)

## Open questions

- Should the cashier see the expected cash before counting? Our suggestion:
  no, the count first, then the result, so the count is honest. Security
  decides.
- The system calls to open and close a shift (backend, in the follow-up).
- Printing a receipt: not in term 1 unless the team asks.

## Changes

- 2026-10-23: Created.
