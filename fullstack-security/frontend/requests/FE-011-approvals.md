---
id: FE-011
title: Approvals (what waits for the owner or the manager)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Approvals (what waits for the owner or the manager)

## Summary

Every action and plan that waits for a yes, in one list: a price change, a
purchase order, a refund, a message to many customers. The same cards also
show in the chat and on Home; this page is where all of them are, with the
history of what was decided.

## Context

- Design: `technical-structure.md` 17.2 (window 8), 6.5 (the planner: one
  approval per plan), 6.6 (how much the AI does alone, the four buttons,
  limits), 18.3 (the owner's morning).
- The frame and how previews work: `FE-005` (section 3).
- Cards: `approval`, `plan` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/approvals`. Owner, and manager for what is under the owner's limit.
  Staff and viewers do not see it
- The sidebar count (`Approvals 1`) is what waits for this person

### 1. Waiting

- Each item: what it does in one line ("Send purchase order #17 to Nile Dairy,
  960 EGP"), who asked (the AI, or a person with the name), when, and the
  area (money, prices, stock, messages, people)
- **An action** (the `approval` card): the change as old → new, or the new
  thing in full; the four buttons **Approve**, **Approve and always allow
  this**, **Decline**, **Never let the AI do this**
- **A plan** (the `plan` card): the reads the AI did, the writes it will do,
  in order, which of them need approval, and **one Approve for all**
  (design 6.5). Each step can be opened to see its change
- **Open** the thing on its page (the purchase order, the item), where it
  shows as waiting (`FE-005`)
- A manager sees items above their limit with "Waiting for the owner." and no
  buttons
- **Approve and always allow this** and **Never let the AI do this** change
  the AI's setting for that area (design 6.6). Before it is saved, one line
  confirms: "From now on the AI will {do this} without asking." / "The AI
  will never {do this}." with **Confirm**

### 2. Decided

History, newest first: what it was, approved or declined, by whom, when, and
the result (done, failed, undone). Filter by approved, declined, by area.

### 3. After a decision

- Approved: the action runs; the item moves to Decided with its result. If it
  fails, it says so and links to the thing
- A plan whose step fails stops; the result says what was done and offers
  **Undo what was done** or **Finish the rest** (design 6.5)

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Step | System call |
|------|-------------|
| What waits, what was decided | Not in the list yet; the route comes in the follow-up |
| Approve, decline | Through the app door (design 2.10); the route comes in the follow-up |
| Always allow, never allow | `ai.set_permission` (owner only), after the confirm line |
| The AI's settings now | `ai.get_permissions` |

### Example responses (draft: the real routes come in a follow-up)

```json
GET approvals (route in the follow-up) {"status": "waiting"}
{"approvals": [
  {"id": "ap_40", "kind": "action", "title": "Send purchase order #17 to Nile Dairy", "area": "money",
   "asked_by": {"type": "ai"}, "asked_at": "2026-10-23T07:00:00.000Z",
   "syscall": "erp.send_purchase_order", "thing": {"type": "purchase_order", "number": 17},
   "changes": [{"field": "status", "before": "draft", "after": "sent"}],
   "amount_cents": 96000, "currency": "EGP", "can_i_approve": true},
  {"id": "pl_5", "kind": "plan", "title": "Offer for slow Tuesday afternoons", "asked_by": {"type": "ai"},
   "reads": ["Sales by hour, last 4 weeks", "Margins on latte and croissant"],
   "steps": [
     {"n": 1, "title": "Create the offer: 20% off, Tuesday 3 to 5 pm", "needs_approval": true},
     {"n": 2, "title": "Tell the barista", "needs_approval": false}
   ], "can_i_approve": true}
]}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Page | Approvals / Waiting / Decided |
| Item | Asked by the AI / Asked by {name} / {time} |
| Areas | Money / Prices / Stock / Messages / People and setup |
| Buttons | Approve / Approve and always allow this / Decline / Never let the AI do this / Approve all / Open |
| Plan | What the AI checked / What it will do / Needs approval |
| Confirm always | From now on the AI will {do this} without asking. / Confirm |
| Confirm never | The AI will never {do this}. / Confirm |
| Manager | Waiting for the owner. |
| Results | Done / Failed / Undone / Undo what was done / Finish the rest |
| Empty | Nothing is waiting for you. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `conflict` | Someone decided it already, or the thing changed | This changed before you approved it. |
| `limit_reached` | Above a manager's limit | This is above your limit. It needs the owner. |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `outside_failed` | Sending to a supplier or a channel failed | It was approved, but sending failed. Try again. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] Waiting shows actions and plans with who asked, the change, and the buttons
- [ ] A plan is approved once for all its steps
- [ ] Always allow and never allow ask for one confirm before saving
- [ ] A manager sees items above their limit without buttons
- [ ] Decided shows the history with the result; a failed plan offers undo or finish
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- The routes to list and decide approvals: backend, with the AI agent team.

## Changes

- 2026-10-23: Created.
