# JEE Battle Royale — Authentication & Pre-Game Flow

A single-file React implementation of the splash → login → difficulty-select → dashboard
flow for the JEE Battle Royale RPG-style auth system.

**Status:** Auth flow is complete and functional. The Dashboard screen is currently a
placeholder — see `PROJECT_REPORT.md` for exactly what's left to wire up.

## Flow

```
Splash → Login (Google sign-in + Remember Me) → Select Difficulty (JEE/NEET/CET)
       → Glitch Transition → Dashboard
```

An existing session (from "Remember Me") skips straight to the glitch transition and dashboard.

## Getting started

```bash
npm install
npm run dev
```

Then open the printed local URL. `npm run build` produces a production build in `dist/`.

## Project structure

```
.
├── index.html
├── src/
│   ├── main.jsx      # Vite entry point, renders <App />
│   ├── index.css     # Tailwind directives
│   └── App.jsx       # Everything: theme, useAuth, all screens, state machine
├── tailwind.config.js
├── postcss.config.js
├── vite.config.js
└── PROJECT_REPORT.md # Full handoff report — read this before making changes
```

## Before you touch the code

Read `PROJECT_REPORT.md`. It covers:
- What each piece of `App.jsx` does
- How authentication and session persistence currently work (mocked, but structured
  for a real Firebase swap — the exact replacement code is commented inline in `useAuth`)
- Exactly what was removed from the Dashboard and the prop contract to reconnect a real one
- Open questions that need to be confirmed with the project owner before continuing
  (notably: whether this should be ported into an existing separate app rather than
  built out standalone — see Section 9 of the report)

## Notes

- Tailwind utility classes are used throughout `App.jsx` for layout; most visual styling
  (theme colors, animations, the HUD frame) is in a template-literal `<style>` block
  injected by each screen component, not Tailwind.
- The login/difficulty background image is embedded directly in `App.jsx` as a base64
  data URI — there is no separate image asset file to manage.
- Icons come from `lucide-react`. Only `Zap`, `RefreshCw`, and `AlertCircle` are currently
  used; add more as the real dashboard needs them.
