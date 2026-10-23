---
id: FE-005
title: The AI panel on every page, previews, and the work panel on Home
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# The AI panel on every page, previews, and the work panel on Home

## Summary

The frame every page of the staff app shares. Each part of the business
(Orders, Menu and stock, Customers, ...) opens as a **full page**, and the
**AI chat stays beside it** in a panel the person can hide. Anything the AI
makes or changes shows as a **preview** until someone approves it. On Home,
previews open in a **work panel** beside the chat. The page requests
`FE-006` to `FE-014` build on this one.

## Context

- Design: `technical-structure.md` section 17.1 ("Pages, the AI panel and the
  work panel", agreed 2026-10-23), 6.5 (the planner: one approval per plan),
  6.6 (Not allowed, Ask first, Act alone; the four buttons of the approval
  card), 17.2 (which windows each role sees).
- Home itself (the sidebar and the chat in the middle): `FE-004`.
- Cards: `app-backend/contracts/cards/` (drafts), above all `approval`,
  `plan` and `question`.
- **The idea:** it works like an AI code editor. The code fills the screen,
  the chat sits beside it, and a change the AI proposes shows as a preview
  until the developer accepts it. Here the page is the business.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### 1. Every part is a full page

Tapping a part in the sidebar opens it as its own page, with its own address.
The sidebar stays (and folds away as on Home).

| Part | Address | Request |
|------|---------|---------|
| Sales screen | `/sales` | `FE-006` |
| Orders | `/orders`, one order `/orders/{number}` | `FE-007` |
| Menu and stock | `/menu` | `FE-008` |
| Customers | `/customers`, one customer `/customers/{id}` | `FE-009` |
| Inbox | `/inbox`, one chat `/inbox/{id}` | `FE-010` |
| Approvals | `/approvals` | `FE-011` |
| Settings | `/settings` | `FE-012` |

A part whose pack is off for the business, or that the person's role does not
allow, does not show in the sidebar. Opening its address shows "You don't
have access to this." (`not_allowed`) or "This part is off for your
business." (`pack_off`), with a link to Home.

### 2. The AI panel

- On **every business page** of the staff app (Home has the chat in the middle
  instead). Not on sign-up, setup or My account
- Sits on the page's **end side**: the right in English, the left in Arabic
- **One button hides it or shows it.** The app remembers the choice per page
  for that person on that device. On the sales screen it is **hidden at
  first** (the cashier needs the room)
- On phones it is a button over the page; tapping it opens the chat over the
  page, with a way back
- It is **the same chat as Home**: the same threads, the **+** menu, the model
  picker and **Ask first / Act alone** (`FE-004`). It shows the current
  thread, with **New request** and **Open on Home** (`/home/t/{thread_id}`)
- **It knows where the person is.** Every message from the panel carries the
  page and what is selected on it, so "mark this one ready" or "why is this
  one losing money?" needs no more words. The panel shows that context above
  the box ("About: Order #1042"), and the person can remove it
- Until the AI is connected, it answers "The AI is not connected yet." (as on
  Home)

### 3. Previews: nothing changes until it is approved

When the AI wants to change something with **Ask first** (or the action
always needs approval):

1. The chat shows the **approval card** (one action) or the **plan card** (a
   plan of several steps, approved once)
2. **On a page**, the thing it changes shows **on the page itself**, marked
   as waiting, with the old and new value: "Latte, medium: 70 → 75 EGP".
   A new thing (a draft purchase order, a new item) shows as a waiting row.
   Tapping the mark opens its card in the AI panel
3. The approval card has four buttons: **Approve**, **Approve and always
   allow this**, **Decline**, **Never let the AI do this** (design 6.6)
4. After **Approve**, the change is made, the mark goes, and the page shows
   the new value. After **Decline**, the mark goes and nothing changed
5. If the thing changed while waiting (someone else edited it), Approve fails
   with `conflict`: show "This changed before you approved it." and the new
   state; the AI can propose again
6. A person whose role cannot approve it sees the card without the buttons
   and "Waiting for the owner."

When the AI acts **alone**, the result shows with **Done by the AI** and,
when the backend says the action can be undone, an **Undo** button.

### 4. The work panel (on Home)

On Home the chat is in the middle, so previews open in the **work panel**
beside it (the end side), only when there is something to show:

- a **preview** waiting for approval (the same as on a page: old → new, or
  the new thing in full, such as a draft purchase order with its lines)
- a **result** after Act alone, with Undo when there is one
- a **thing the person opened** from a card in the chat (an order, a recipe,
  a customer), with **Open full page**

It holds **only the app's own cards and pages** (the `contracts/cards/`
list), never content the model made up. One button closes it. On phones it
opens over the chat, with a back button.

### 5. Messages carry the page

Draft; the full message and streaming format comes with the AI agent team
(the open point in `FE-004`):

```json
POST (route in the follow-up)
{
  "thread_id": "t_8f2k",
  "text": "make this one ready",
  "context": {"page": "orders", "selected": {"type": "order", "number": 1042}},
  "model": "auto",
  "mode": "ask_first"
}
```

An approval card in an answer:

```json
{
  "type": "approval",
  "id": "ap_31",
  "title": "Change the price of Latte, medium",
  "asked_by": {"type": "ai"},
  "syscall": "erp.change_price",
  "changes": [
    {"thing": {"type": "item", "id": "it_7", "label": "Latte, medium"},
     "field": "price_cents", "before": 7000, "after": 7500}
  ],
  "can_undo": true,
  "can_i_approve": true,
  "status": "waiting"
}
```

A result after Act alone:

```json
{"type": "result", "title": "Logged 2 liters of milk as waste", "by": "ai", "undo": {"available": true, "until": "2026-10-23T22:00:00.000Z"}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Panel button | Ask the AI / Hide the AI |
| Context line | About: {thing} |
| Open on Home | Open on Home |
| Waiting mark | Waiting for approval |
| Approval buttons | Approve / Approve and always allow this / Decline / Never let the AI do this |
| Role cannot approve | Waiting for the owner. |
| Done alone | Done by the AI |
| Undo | Undo / Undone. |
| Work panel | Open full page / Close |
| Changed while waiting | This changed before you approved it. |
| Pack off | This part is off for your business. |
| AI not connected | The AI is not connected yet. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_logged_in` | No session, or it ended | You were logged out. Please log in again. (then `/login`) |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `pack_off` | The part's pack is off | This part is off for your business. |
| `needs_approval` | The action now waits for approval | Sent for approval. |
| `conflict` | The thing changed before Approve or Undo | This changed before you approved it. |
| `limit_reached` | Above the person's limit (a manager) | This is above your limit. It needs the owner. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] Each part opens as a full page at the addresses above, with the sidebar
- [ ] Every business page has the AI panel on the end side (right in English, left in Arabic)
- [ ] One button hides and shows it; the choice is remembered per page; hidden at first on the sales screen
- [ ] On phones the panel opens over the page with a way back
- [ ] The panel uses the same threads as Home, with New request and Open on Home
- [ ] A message from the panel carries the page and the selected thing, shown as "About: ..." and removable
- [ ] A waiting change shows on the page with old → new; Approve, Decline and the other two buttons work against the example responses
- [ ] Act alone results show "Done by the AI" and Undo when the backend allows it
- [ ] On Home, the work panel opens only with something to show, and only with the app's own cards
- [ ] Works in Arabic (right to left) and English, on phone, tablet and laptop

## Open questions

- The message and streaming format, and the route to approve or decline:
  with the AI agent team, in the follow-up.
- Who writes the Arabic texts (same open point as `ALL-001`)?

## Changes

- 2026-10-23: Created.
