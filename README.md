# team-handoffs

One place where teams hand off requirements to each other and keep informal notes.

This is **not** formal documentation. It's a shared notebook: what we asked each other for, what we agreed on, and why.

## Teams

| Group | Team | Code | Folder |
|---|---|---|---|
| Full stack & security | Backend | `BE` | `fullstack-security/backend/` |
| Full stack & security | Frontend | `FE` | `fullstack-security/frontend/` |
| Full stack & security | Security | `SEC` | `fullstack-security/security/` |
| AI | AI Agent | `AGT` | `ai/agent/` |
| AI | AI Data | `DATA` | `ai/data/` |
| Everyone | Shared | `ALL` | `shared/` |

## Folders

Each team folder has:

- `requests/` for things **other teams need from this team**. The team that has to do the work owns the folder.
- `notes/` for the team's own informal docs (how something works, decisions, gotchas).
- `README.md` with who to contact and what to include when you send them something.

Other folders:

- `fullstack-security/frontend/designs/` has the frontend team's designs, one folder per request (`FE-003-business-setup-steps/`), as pictures plus a link to the design file. The format is in its `README.md`.
- `templates/` has the files to copy when you start something new.
- `shared/` is for things that belong to everyone: decisions all teams must follow, and requests with no single receiving team. Use `ALL` as the code and `to: everyone`.

## How to send a request to another team

1. Copy `templates/request.md`.
2. Put it in the **receiving team's** `requests/` folder.
3. Name it `<CODE>-<number>-<short-name>.md`, using the receiving team's code and their next number.
   Example: backend asks frontend for something → `fullstack-security/frontend/requests/FE-001-auth-flow-texts.md`
4. Fill in the header and the sections. Commit and push.
5. Send the file link to the team on WhatsApp or Discord.

## When something changes

Discussion happens on WhatsApp or Discord. When it leads to a change:

1. Edit the file.
2. Update `updated:` in the header.
3. Add one line under **Changes** at the bottom of the file saying what changed and who agreed.

That way the file always shows the latest agreement, and the Changes list shows how we got there.

## Status

Set `status:` in the header to one of:

| Status | Meaning |
|---|---|
| `open` | Sent, waiting for the receiving team |
| `in-progress` | Receiving team is working on it |
| `done` | Finished |
| `dropped` | Not doing it (write why in Changes) |

The receiving team updates the status.

## Finding things

- All open requests for your team: search your `requests/` folder for `status: open`.
- Everything from one team: search for `from: backend` (or any team).
