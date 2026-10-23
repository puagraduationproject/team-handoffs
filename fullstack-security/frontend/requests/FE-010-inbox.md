---
id: FE-010
title: Inbox (customer chats: the AI or a person, take over, hand back)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-23
updated: 2026-10-23
author: backend
---

# Inbox (customer chats: the AI or a person, take over, hand back)

## Summary

Every customer chat from the website chat and Telegram, in one place. The AI
answers most of them alone; the staff watch, take over when they want or
when the AI passes a chat to them, reply, and hand it back.

## Context

- Design: `technical-structure.md` 17.2 (window 7), 10.1 and 10.2
  (Channels), 18.1 and 18.2 (a customer orders, a customer complains), 6.1
  (the customer agent hands off), 7.3 (packs `telegram`, `whatsapp` later).
- The frame (full page, the AI panel, previews): `FE-005`.
- Cards the AI sends customers (shown in the chat): `order`, `item`,
  `question` in `app-backend/contracts/cards/`.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where and who

- `/inbox`, one chat at `/inbox/{id}`. Always on (website chat is in the
  essentials)
- Owner, manager and staff; a viewer read-only

### 1. The list of chats

- Each chat: the customer's name (or "New customer"), the channel (website
  chat, Telegram), the last message and when, unread, the state (open,
  waiting, closed), **who handles it: the AI or a person** (with the name),
  and tags
- **Needs a person** first: chats the AI passed to staff, with the reason in
  one line ("Asked for a refund", "Angry about a cold latte")
- **Filters:** needs a person, handled by the AI, mine, by channel, closed
- New messages arrive without reloading; the sidebar count updates
  (`Inbox 2`)

### 2. One chat

- **The messages**, with who sent each: the customer, the AI, or a staff
  member (with the name). The cards the AI sent (an order, an item, a
  question with choices) show as cards. Voice notes show their text when
  there is one; images show
- **At the top, when the AI passed it:** "The AI passed this to staff:
  {reason}"
- **Take over:** the person handles it from now; the AI stops answering this
  chat. **Hand back to the AI** gives it back
- **Reply box** (only while a person handles it): text, and later images.
  While the AI handles it, the box says "The AI is answering. Take over to
  reply."
- **Assign** to a staff member, **Tags**, **Close**
- **Beside the chat:** the customer (the `customer` card: usual order,
  points, open cases) with **Open customer**, and their open order with
  **Open order**

### 3. The AI on this page

Two different AIs meet here, and the page must keep them apart:

- The **customer agent** talks to the customer inside the chat. Staff do not
  type to it
- The **AI panel** (`FE-005`) is the owner partner, for the staff: "sum up
  this chat", "what should I answer?", "write a polite reply". A reply it
  drafts goes into the reply box for the person to send; it never sends to
  the customer by itself from here

### System calls used (draft pages in `app-backend/contracts/syscalls/`)

| Step | System call |
|------|-------------|
| The list | `channels.list_conversations` |
| One chat | `channels.get_conversation` |
| Reply | `channels.reply` |
| Take over | Not in the list yet (today `channels.handoff_to_staff` is the AI's); the name comes in the follow-up |
| Hand back | `channels.hand_back_to_ai` |
| Assign, tags, close | `channels.assign_conversation`, `channels.tag_conversation`, `channels.close_conversation` |
| The customer beside it | `crm.get_customer`, `erp.list_orders` (by customer, open) |
| New messages arriving | Event `MessageReceived`; how the app hears it comes in the follow-up |

### Example responses (draft: the real routes come in a follow-up)

```json
channels.list_conversations {"filter": "needs_person"}
{"conversations": [{"id": "cv_9", "customer": {"id": "cu_3", "name": "Mariam"}, "channel": "telegram",
  "state": "open", "handled_by": {"type": "user", "name": null}, "handoff_reason": "Angry about a cold latte",
  "last_message": {"body": "My latte was cold and late", "at": "2026-10-23T09:40:00.000Z"}, "unread": 1, "tags": []}]}

channels.get_conversation {"id": "cv_9"}
{"conversation": {"id": "cv_9", "state": "open", "handled_by": {"type": "ai"}, "messages": [
  {"id": "m_1", "direction": "in", "sender": {"type": "customer"}, "type": "text", "body": "2 lattes and a croissant, takeaway", "at": "2026-10-23T09:10:00.000Z"},
  {"id": "m_2", "direction": "out", "sender": {"type": "ai"}, "type": "card", "card": {"type": "order", "number": 1042, "total_cents": 21500, "currency": "EGP"}, "at": "2026-10-23T09:10:05.000Z"},
  {"id": "m_3", "direction": "in", "sender": {"type": "customer"}, "type": "audio", "body": null, "transcript": "Is it ready?", "at": "2026-10-23T09:25:00.000Z"}
]}}
```

## Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Page | Inbox |
| Filters | Needs a person / Handled by the AI / Mine / Website chat / Telegram / Closed |
| Chat | New customer / The AI / {name} / Open / Waiting / Closed |
| Handoff | The AI passed this to staff: {reason} |
| Actions | Take over / Hand back to the AI / Assign / Tags / Close / Open customer / Open order |
| Reply box | Write a reply... / Send |
| AI answering | The AI is answering. Take over to reply. |
| Empty | No chats yet. Connect Telegram or share your shop page. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `conflict` | Someone else took over or closed it | {name} is handling this chat now. |
| `outside_failed` | Telegram did not take the message | The message did not go through. Try again. |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| `not_found` | The chat is gone | We couldn't find this chat. |
| `not_logged_in` | No session | You were logged out. Please log in again. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] The list shows every chat with who handles it, "Needs a person" first with the reason
- [ ] One chat shows messages from the customer, the AI and staff, with the AI's cards as cards
- [ ] Take over, reply, hand back, assign, tag and close work against the example responses
- [ ] The reply box is only open while a person handles the chat
- [ ] A reply drafted by the AI panel goes into the box, never straight to the customer
- [ ] New messages show without reloading
- [ ] Phone, tablet, laptop; Arabic (right to left) and English; the AI panel from `FE-005`

## Open questions

- The system call for a person to take over (backend, in the follow-up).
- How the app hears new messages live: backend, in the follow-up.

## Changes

- 2026-10-23: Created.
