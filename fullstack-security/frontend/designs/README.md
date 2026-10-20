# Designs

Where the frontend team shares its designs with the other teams: backend,
security and the AI teams look here to see what a screen will show and which
data it needs.

The look is the frontend team's choice. Other teams comment on **what** a
screen shows and does (data, steps, texts, errors), not on how it looks.

## One folder per request

Name it after the request it answers, so everyone can find both:

```
designs/
├── FE-002-landing-and-sign-in/
│   ├── README.md
│   ├── landing-laptop-ar.png
│   ├── landing-phone-ar.png
│   ├── login-email-phone-ar.png
│   └── ...
├── FE-003-business-setup-steps/
└── FE-004-home-chat-shell/
```

Designs with no request yet (the foundations, a new idea) go in a folder with
a short name of their own, for example `foundations/`.

## What goes in each folder

1. **`README.md`**, short:
   - the request it answers (`FE-003`) and its state: `draft`, `for review`
     or `final`
   - **the link to the design file** (Figma, or the tool you use), with view
     access for the whole team
   - the list of screens, in the order of the flow
   - what is still open, and questions for the other teams
2. **Pictures of every screen**, so people can see them without logging in
   anywhere:
   - **PNG** (or WebP), exported at 2x, each under 2 MB
   - the **phone** size for every screen; the **laptop** size too for the
     landing page and anything with a sidebar
   - **Arabic (right to left) first**; English too when the layout changes
   - the states that matter: empty, loading, an error, the first day
3. **A flow picture** when there are steps (`flow.png`): the screens with
   arrows, including where errors go back to

## File names

`<screen>-<state>-<size>-<language>.png`, small letters and dashes. Leave out
the state when it is the normal one.

| Part | Values |
|------|--------|
| screen | `landing`, `login`, `setup-name`, `home`, ... (the step names in the request) |
| state | `empty`, `loading`, `error`, `first-day`, ... |
| size | `phone`, `tablet`, `laptop` |
| language | `ar`, `en` |

Examples: `setup-handle-error-phone-ar.png`, `home-first-day-laptop-en.png`.

## What not to put here

- The design tool's own files (`.fig`, `.psd`, `.sketch`, `.xd`): keep them in
  the tool and link them in the README
- Videos larger than 10 MB: link them instead
- Real people's data in the pictures (use made-up names, emails and phones)

## When it changes

Replace the pictures (git keeps the old ones), update the state in the
folder's `README.md`, and add one line under **Changes** there, as with
requests.
