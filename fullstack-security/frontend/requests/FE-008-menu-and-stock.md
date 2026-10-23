---
id: FE-008
title: Menu and stock (items, recipes and cost, stock and waste, suppliers and purchases)
from: backend
to: frontend
cc: [agent]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Menu and stock (items, recipes and cost, stock and waste, suppliers and purchases)

## Summary

Where the owner and the manager keep what the business sells and what it
uses: the menu with sizes, options and prices; the recipe and real cost of
each item; the stock, waste and counts; the suppliers and purchase orders.
One page with four parts. Each part shows only when its pack is on.

## Context

- Design: `technical-structure.md` 17.2 (window 5), 8.1 (what ERP owns), 8.5
  (ERP system calls), 1.1 and 1.2 (the problems: real cost of a cup, waste,
  running out at rush hour, reordering by memory, expiry), 7.3 (packs).
- The frame (full page, the AI panel, previews): `FE-005`.
- Cards: `item`, `stock` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/menu`, with the four parts at `/menu/items`, `/menu/recipes`,
  `/menu/stock`, `/menu/purchases`; one item at `/menu/items/{id}`
- **Owner and manager:** everything. **Staff:** stock and waste only (no
  costs, no prices to change, no suppliers). **Viewer:** read-only

### 1. Menu (always on: the catalog)

- **Items by category**, with name, photo, price (or "from 70" with sizes),
  and **available** or **sold out**. Search; filter by category
- **One item:** name, description, photo, category, sizes with their price,
  option groups (pack `options`: milk type, extra shot, sugar, each with its
  price), and the fields the template adds (coffee shop: size, milk type;
  small market: brand, unit, barcode, expiry). The backend sends which fields
  this business has
- **Add item**, **Edit**, **Sold out today / Back in stock**, **Hide** (archive,
  can be restored)
- **Change price** always needs approval: the new price shows as waiting
  ("70 → 75 EGP") until the owner approves (`FE-005`)
- **Categories:** add, rename, order them
- **Add from a menu photo:** opens the AI panel with the photo, and the items
  the AI reads show as waiting rows to approve (the first-day step in
  `FE-004`)

### 2. Recipes and cost (pack `recipes`)

- **Ingredients:** name, unit (g, ml, piece), cost per unit, supplier
- **The recipe of each item and size:** which ingredients and how much one sale
  uses
- **Cost and margin** per item: cost now, price, margin in pounds and percent,
  and the change since last month. **Losing money** or a falling margin is
  marked
- **Suggest a price** for a target margin (the AI can also do it); applying it
  is a price change (approval)

### 3. Stock (pack `stock`)

- **Levels:** each ingredient or product with what is on hand, the unit, and
  **Low** or **Runs out {day}** at the current pace. Filters: low, expiring
  (pack `expiry`), all
- **Log waste:** what, how much, why (spoiled, spilled, expired, free drink,
  other). Staff use it often: it must be quick
- **Count:** type the real amount; the difference is a correction. Above a
  limit it needs approval
- **Stock in** (bought without a purchase order): what, how much, the cost
- **History** of one ingredient: every movement (sale, waste, count, purchase)
  with who and when

### 4. Suppliers and purchases (pack `purchasing`)

- **Suppliers:** name, phone, what they supply and at what price, delivery
  days. **Compare prices** of one ingredient across suppliers
- **Purchase orders** by status: draft, approved, sent, received, cancelled;
  each with its number, supplier, lines and total
- **New purchase order**, or **from low stock** (a draft with what is running
  out). Edit the lines; **Send** (above a limit needs approval); **Receive**
  (full or part, with the real costs; this adds the stock and updates the
  costs); **Cancel**
- A purchase order the AI drafted shows as a waiting row (the morning card
  "Milk runs out tomorrow [Order now]" leads here)

### 5. The AI on this page

The AI panel (`FE-005`) knows the selected item, ingredient or purchase
order: "what does a latte cost me now?", "log 2 liters of milk as spoiled",
"order milk for the week". What it proposes shows as waiting on the page.

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Part | System calls |
|------|--------------|
| Menu | `erp.list_categories`, `erp.search_items`, `erp.get_item`, `erp.create_item`, `erp.update_item`, `erp.change_price` (always approval), `erp.set_availability`, `erp.manage_modifiers`, `erp.archive_item` |
| Recipes and cost | `erp.list_ingredients`, `erp.create_ingredient`, `erp.update_ingredient`, `erp.get_recipe`, `erp.set_recipe`, `erp.get_item_cost`, `erp.suggest_price` |
| Stock | `erp.get_stock_levels`, `erp.forecast_stock`, `erp.log_waste`, `erp.adjust_stock`, `erp.record_stock_in`, `erp.get_stock_history` |
| Suppliers and purchases | `erp.list_suppliers`, `erp.get_supplier`, `erp.create_supplier`, `erp.update_supplier`, `erp.compare_supplier_prices`, `erp.draft_purchase_order`, `erp.update_purchase_order`, `erp.send_purchase_order`, `erp.receive_purchase`, `erp.cancel_purchase_order` |

### Example responses (draft: the real routes come in a follow-up)

```json
erp.get_item {"id": "it_7"}
{"item": {"id": "it_7", "name": "Latte", "category": {"id": "c_1", "name": "Hot drinks"}, "available": true,
  "sizes": [{"id": "v_1", "name": "Medium", "price_cents": 7000}],
  "fields": [{"key": "milk_type", "label": "Milk type", "value": "Full cream"}],
  "cost": {"cost_cents": 2350, "margin_cents": 4650, "margin_percent": 66, "change_since_last_month_cents": 300}}}

erp.get_stock_levels {"filter": "low"}
{"levels": [{"stock_ref": {"type": "ingredient", "id": "in_2", "name": "Milk"}, "on_hand": 2000, "unit": "ml",
  "runs_out_on": "2026-10-24", "low": true}]}

erp.log_waste {"stock_ref": {"type": "ingredient", "id": "in_2"}, "qty": 2000, "reason": "spoiled"}
{"movement": {"id": "sm_88", "type": "waste", "qty": 2000, "unit": "ml"}, "undo": {"available": true}}

erp.draft_purchase_order {"from": "low_stock"}
{"purchase_order": {"number": 17, "status": "draft", "supplier": {"id": "su_1", "name": "Nile Dairy"},
  "lines": [{"name": "Milk", "qty": 24000, "unit": "ml", "unit_cost_cents": 4, "line_total_cents": 96000}],
  "total_cents": 96000, "currency": "EGP"}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Parts | Menu / Recipes and cost / Stock / Suppliers and purchases |
| Menu | Add item / Edit / Sold out today / Back in stock / Hide / Restore / Change price / Categories / Add from a menu photo / from {price} |
| Item | Name / Description / Photo / Category / Sizes / Options / Price |
| Recipes | Ingredients / Unit / Cost per unit / Recipe / Cost / Margin / Since last month / Losing money / Suggest a price / Target margin |
| Stock | On hand / Low / Runs out {day} / Expiring / Log waste / Count / Stock in / History |
| Waste reasons | Spoiled / Spilled / Expired / Free drink / Other |
| Purchases | Suppliers / Purchase orders / New purchase order / From low stock / Draft / Approved / Sent / Received / Cancelled / Send / Receive / Compare prices / Delivery days |
| Waiting | Waiting for approval |
| Empty menu | Nothing here yet. Add an item or send a photo of your menu. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `needs_approval` | A price change, a count or a purchase order above the limit | Sent for approval. |
| `invalid_input` | A missing or wrong field (the details name it) | Check {field}. |
| `conflict` | The item or purchase order changed meanwhile | This changed. Here is the latest. |
| `not_allowed` | The role does not allow it (staff and prices) | You don't have access to this. |
| `pack_off` | The part's pack is off | This part is off for your business. |
| `not_found` | The item is gone | We couldn't find this item. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] The four parts work against the example responses, each hidden when its pack is off
- [ ] Items with sizes, options and the template's fields can be added, edited, marked sold out and hidden
- [ ] A price change shows as waiting until approved
- [ ] Recipes give the cost and margin per item; losing money is marked
- [ ] Waste is quick to log; counts and stock in work; history shows every movement
- [ ] Purchase orders go from draft to received; receiving updates stock and costs
- [ ] Staff see only stock and waste
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- Units: the backend stores the smallest unit (ml, g); the screens may show
  liters and kilos. Frontend's choice, as long as the value sent back is
  exact.

## Changes

- 2026-10-23: Created.
