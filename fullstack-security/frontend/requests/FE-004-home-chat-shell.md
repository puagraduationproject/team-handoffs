---
id: FE-004
title: Home, the first page after log-in (the chat and the sidebar)
from: backend
to: frontend
cc: [agent, security]
status: open
priority: high
created: 2026-10-21
updated: 2026-10-21
author: backend
---

# Home, the first page after log-in (the chat and the sidebar)

## Summary

The first page a person sees after logging in (and after the business setup,
`FE-003`) is **Home**: a sidebar on the left and the AI chat in the middle,
like a chat app. Please build its shell: the sidebar, the middle with the
cards and the chat box, and the first-day version. The data behind it is not
all built yet; build with the example responses below, and we send the real
routes in a follow-up.

## Context

- Design: `technical-structure.md` section 17.1 (the layout and the first
  page after log-in), 1.5 (Home is a to-do list), 7.7 (what setup leaves for
  later), 6.2 (the model picker), 17.2 (which windows each role sees).
- Cards the AI answers with: `app-backend/contracts/cards/` (drafts).
- **Reference for the flow and the structure only:** the Claude app
  (claude.ai) after log-in: a sidebar that folds away, and one chat box in the
  middle with the choices under it. Not for how it looks.

## Yours to decide

**Colors, palette, fonts, style, spacing and the exact layout are the frontend
team's choice.** Nothing in this request sets them. Follow your own
foundations (`shared/ALL-001-frontend-foundations.md`).

## Requirements

### Where

- **`/home`** on the test address. A logged-in person who opens `/` or
  `/login` goes there; one with no business yet goes to `/setup` (`FE-003`).
  Later the app moves to its own address, with Home at `/`
- A thread (one conversation) gets its own address, for example
  `/home/t/{thread_id}`, so it can be reloaded and shared inside the business

### The sidebar

Folds away with one button. On phones it is closed by default and opens over
the page.

1. **Business switcher** at the top: the business name ("Nova Café ▾"). Opens
   the list of the person's businesses (`GET /api/app/businesses`). With one
   business it still shows the name
2. **+ New request**: an empty chat
3. **Search**: searches the person's threads
4. **The parts of the business**, each with a count when there is something
   waiting. Only what the person's role allows (design 17.2). For the owner:
   Inbox, Orders, Approvals, Customers, Menu and stock, Sales screen. Show the
   first five; the rest under **More**. These pages are not built yet: the
   links can go to a simple "coming soon" page
5. **Pinned** and **Recent threads**
6. **The person**, at the bottom: their name and `username@handle · Role`
   (`ahmed@nova · Owner`). Opens a menu: **My account** (`/account`, already
   built), **Settings** (coming soon), **Log out** (`POST /api/auth/logout`,
   then `/`)

### The middle

1. **Greeting:** "Good morning, {name}" (morning, afternoon or evening, in the
   business's time zone, Cairo for now)
2. **Cards, at most 3:** what needs the person today. Each card has a title,
   one line, and up to two buttons ("Milk runs out tomorrow" with **Order
   now**). With no cards, nothing is shown: only the greeting and the box
3. **The chat box:** "Ask or tell me anything..." with:
   - **+** menu: Upload a photo (menu, invoice), and later quick actions
   - **The model picker:** shows **Auto** by default (our own model). Opens a
     list of models; each says where it runs and whether data leaves our
     server (design 6.2). Only Auto works for now; show the others as "soon"
   - **Ask first / Act alone:** a switch; Ask first by default
   - **Voice** button: shown, "soon"
4. **Answers** stream in: text first, then cards when they come (design 17.1).
   An unknown card shows its plain text

### The first time (right after setup)

The business is empty, so Home helps finish the setup in the chat:

- The greeting: "Welcome, {name}. {Business} is ready." and under it "Let's
  add what you sell first."
- **Quick buttons** under the chat box: **Send a photo of your menu**, **Add a
  product**, **Connect Telegram**. Pressing one starts that step in the chat.
  Each button goes away once its step is done (the backend says which are
  left, see below)

### Example responses (draft: the real routes come in a follow-up)

```json
GET /api/app/home?business=nova
{
  "greeting_name": "Ahmed",
  "first_day": true,
  "cards": [
    {"type": "stock", "title": "Milk runs out tomorrow", "line": "2 liters left, you use 6 a day", "actions": [{"key": "order_milk", "label": "Order now"}]}
  ],
  "quick_actions": [
    {"key": "menu_photo", "label": "Send a photo of your menu"},
    {"key": "add_product", "label": "Add a product"},
    {"key": "connect_telegram", "label": "Connect Telegram"}
  ],
  "counts": {"inbox": 2, "orders": 4, "approvals": 1},
  "model": {"key": "auto", "label": "Auto"},
  "mode": "ask_first"
}

GET /api/app/threads?business=nova
{"threads": [{"id": "t_8f2k", "title": "Milk order for Thursday", "pinned": false, "updated_at": "2026-10-21T08:12:00.000Z"}]}

GET /api/app/businesses     (live now)
{"businesses": [{"name": "Nova Café", "handle": "nova", "business_type": "coffee_shop", "team_size": "alone", "role": "owner", "username": "ahmed", "display_name": "Ahmed Nassem"}]}
```

Sending a message and the streamed answer: the format comes with the
follow-up, together with the AI agent team. Until then the box can show the
message and a fixed answer ("The AI is not connected yet").

### Texts (English; Arabic is the default language and still needed)

| Where | Text |
|-------|------|
| Greeting | Good morning, {name} / Good afternoon, {name} / Good evening, {name} |
| First day, line 1 | Welcome, {name}. {Business} is ready. |
| First day, line 2 | Let's add what you sell first. |
| Chat box | Ask or tell me anything... |
| New | New request |
| Search | Search |
| More | More |
| Pinned / Recent | Pinned / Recent |
| Parts | Inbox, Orders, Approvals, Customers, Menu and stock, Sales screen |
| Person menu | My account / Settings / Log out |
| Model picker | Auto (our model) / Soon |
| Mode | Ask first / Act alone |
| Quick buttons | Send a photo of your menu / Add a product / Connect Telegram |
| Coming soon page | This part is coming soon. |
| AI not connected | The AI is not connected yet. |

## Error codes

| Code | When | Message shown to user |
|---|---|---|
| `not_logged_in` | No session, or it ended | You were logged out. Please log in again. (then `/login`) |
| `not_allowed` | The role does not allow it | You don't have access to this. |
| — | Network down | We couldn't reach the server. Check your connection. |
| — | Anything else | Something went wrong. Please try again. |

## Done when

- [ ] `/home` shows the sidebar and the middle as above; `/` and `/login` take a logged-in person there
- [ ] The sidebar folds away; closed by default on phones
- [ ] The business switcher lists the person's businesses from `GET /api/app/businesses`
- [ ] The person menu has My account, Settings and Log out, and Log out works
- [ ] The middle shows the greeting, up to 3 cards and the chat box with +, the model picker, Ask first / Act alone
- [ ] The first-day version shows its greeting and the quick buttons
- [ ] Built against the example responses; easy to switch to the real routes
- [ ] Works on phone, tablet and laptop, in Arabic (right to left) and English
- [ ] Built in `app-web/staff/home/`

## Open questions

- Who writes the Arabic texts (same open point as ALL-001)?
- The message and streaming format: with the AI agent team, in the follow-up.

## Changes

- 2026-10-21: Created.
