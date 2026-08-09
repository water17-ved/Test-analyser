# JEE BATTLE ROYALE — Authentication & Pre-Game Flow
## Handoff Report

**Deliverable:** `/mnt/user-data/outputs/App.jsx`
**Status:** Auth flow complete and functional. Dashboard intentionally removed — to be supplied and wired in by the receiving agent.

---

## 1. WHAT THIS FILE DOES

A single-file React app implementing everything **before** the real game dashboard:

```
Splash → Login → Select Difficulty (JEE/NEET/CET) → Glitch Transition → Dashboard*
```
`*Dashboard` is currently a minimal placeholder — see Section 5.

On load, an existing session (if "Remember Me" was used) skips Splash + Login + Difficulty and goes straight to the Glitch Transition, then the Dashboard.

---

## 2. VISUAL DESIGN

- **Reference:** a supplied HUD-style image (dark blue/black background, cyan neon circuit lines, glowing frame) is the visual source of truth for Login and Difficulty screens. The image is embedded directly in the file as a compressed base64 data URI (`LOGIN_BG_URL`) so the artifact is self-contained — no external asset dependency.
- **Background treatment:** `cover` sizing (fits the viewport without cropping), with a dark radial+flat gradient overlay on top for text legibility.
- **Notification panel (Login screen):** wide landscape panel (not a narrow card), 50% transparent so the background shows through, blurred slightly (3px) so the circuit linework stays visible but doesn't fight with the text. Bordered by a crisp thin white/cyan line plus an outer glowing HUD frame (angled top/bottom bars, thin integrated side rails — deliberately subtle, not a bold box).
- **Typography:** Orbitron (display/titles), Rajdhani (labels/UI text).
- **Two theme sets:**
  - `AUTH_THEME` — cyan HUD palette, used by Splash, Login, Glitch, Difficulty.
  - Dashboard previously had its own gold/violet theme; removed along with the dashboard (see Section 5).

---

## 3. COMPONENTS (CURRENT FILE CONTENTS, TOP TO BOTTOM)

| Component | Purpose |
|---|---|
| `LOGIN_BG_URL` | Base64 data URI of the background art |
| `AUTH_THEME` | Shared CSS for all auth-flow screens (Splash/Login/Glitch/Difficulty) |
| `GoogleGIcon` | Inline SVG Google "G" mark for the sign-in button |
| `useAuth()` | Single authentication abstraction — see Section 4 |
| `SplashScreen` | 2.5s cinematic boot sequence with progress bar |
| `LoginScreen` | Reference-image login UI, Google sign-in, Remember Me checkbox, 4 states (default/loading/error/success) |
| `GlitchTransition` | ~0.85s RGB-split glitch effect, text "ENTERING DUNGEON" |
| `DifficultySelect` | New — lets the player pick JEE / NEET / CET before entering |
| `Dashboard` | **Placeholder only** — see Section 5 |
| `App` | Root state machine tying it all together |

---

## 4. AUTHENTICATION (`useAuth`)

Single hook; every component talks to auth only through it. Currently **mocked** (simulated 1.4s delay, ~8% random failure to exercise the error state) but structured for a drop-in Firebase replacement — the exact code to swap in is commented inline at the mock block.

**Session persistence — this is real, not mocked:**
- "Remember Me" checked → user profile saved to `localStorage` (`jbr_persistent_v1`) → survives browser restart
- "Remember Me" unchecked → saved to `sessionStorage` (`jbr_session_v1`) → cleared when tab closes
- Only `{ uid, displayName }` is ever stored — never a password, token, or OAuth secret
- On app mount, `useAuth` checks for either and restores the session automatically

**Interface:**
```js
{
  status,          // 'checking' | 'signed-out' | 'loading' | 'error' | 'signed-in'
  user,            // { uid, displayName } | null
  error,           // string | null
  rememberPlayer,  // boolean
  setRememberPlayer,
  signIn,          // () => void — replace internals with real Firebase call
  signOut,
  retry,
}
```

---

## 5. DASHBOARD — REMOVED (ACTION REQUIRED BY NEXT AGENT)

The previous JEE-Battle-Royale dashboard (player level/XP/rank, quote banner, dungeon-map nav tiles, streak counter, etc.) has been **fully deleted** from this file, along with its dedicated theme (`DASHBOARD_THEME`), its data hook (`usePlayerData`), and every sub-component that only it used (`QuoteBanner`, `XPBar`, `RankBadge`, `AimCard`, `NavTile`, `NAV_TILES`, `DashboardSkeleton`, `DashboardError`, `EmptyProgress`). None of that code exists in the file anymore — it is not commented out, it is gone.

In its place is a minimal placeholder with the **same prop contract**, so it's a drop-in swap:

```js
function Dashboard({ user, examTrack, onLogout }) {
  // shows player name, chosen track, and a working sign-out button
}
```

**To reconnect a real dashboard:** replace this function with your implementation, keeping the same three props:
- `user` — `{ uid, displayName } | null`, sourced from `useAuth()`
- `examTrack` — `'jee' | 'neet' | 'cet' | null`, sourced from the Difficulty screen
- `onLogout` — call to sign out and return to the auth flow (already wired to `auth.signOut()` + reset state)

Nothing in `App`'s state machine needs to change to accept a new `Dashboard` — it's already passing the right props.

Icon imports were also trimmed: only `Zap`, `RefreshCw`, `AlertCircle` remain (used by Splash/Login/Glitch). Re-add whatever `lucide-react` icons your new dashboard needs.

---

## 6. APP STATE MACHINE

```js
const APP_STATE = {
  SPLASH: 'splash',
  LOGIN: 'login',
  DIFFICULTY: 'difficulty',
  TRANSITION: 'transition',
  DASHBOARD: 'dashboard',
};
```

```
START
  ↓
Check existing session (useAuth, on mount)
  │
  ├─ signed-in  → TRANSITION → DASHBOARD
  └─ signed-out → SPLASH → LOGIN → DIFFICULTY → TRANSITION → DASHBOARD
```

`examTrack` (the difficulty choice) is held in `App`'s own state and passed down to `Dashboard`; it resets to `null` on logout.

---

## 7. ACCESSIBILITY

- Google sign-in button and difficulty tiles are real `<button>`s, keyboard-focusable, with `aria-label`/`aria-busy` as appropriate
- Remember Me checkbox is a native `<input type="checkbox">`
- Error states use `role="alert"` + `aria-live="assertive"`
- Success/glitch states are `aria-hidden` (decorative/transitional, non-interactive)
- All custom animations respect `prefers-reduced-motion`

---

## 9. ORIGIN CONTEXT (RECONCILED FROM A PARALLEL SESSION)

Another session produced a report describing the same effort but with several details that don't match this thread's actual history. That report is **not discarded** — it surfaces one important fact this thread didn't otherwise have — but its specifics are corrected below rather than passed through verbatim, since it was explicitly flagged as outdated.

**The one fact worth carrying forward:**
> There may be a real, already-existing "JEE Battle Royale" app — a single-file vanilla HTML/CSS/JS PWA (`index.html`) — that already has its own splash screen, its own working Firebase Google Sign-In (used for optional leaderboard sync; progress stays local), and multiple existing visual themes (Fortnite, COD, Shadow Fight, Minimal).

**This is unverified from this thread's side** — no `index.html` was uploaded here, and nothing in this conversation confirms or references it. If true, it changes the integration picture significantly: `App.jsx` would not be extending a from-scratch project, but would need to be **ported into an existing vanilla-JS codebase that already has real Firebase auth**, rather than having its own mock `useAuth()` swapped for Firebase later. The receiving agent should confirm this with the user before assuming either scenario.

**Corrections to the parallel report's specifics** (these do not describe `App.jsx`):
| Parallel report claimed | Actually true in this thread |
|---|---|
| No `Dashboard.jsx` was ever uploaded | A `Dashboard.jsx` **was** uploaded early in this thread and used as the basis for the original dashboard (later removed per the user's request — see Section 5) |
| File named `JBR-Auth-Dashboard.jsx` | File is `App.jsx` |
| Persistence via Claude-artifact `window.storage` API | Persistence via standard `localStorage`/`sessionStorage` (see Section 4) — no `window.storage` calls exist in this file |
| ~12% simulated auth failure rate | ~8% in this file's `useAuth()` |
| Syntax-checked with esbuild | No esbuild or build-tool check has been run in this thread |
| Dashboard placeholder shows stat chips + full card grid (Dungeon Map, Tests, Papers, Leaderboard, Achievements, Analytics, Uploads) | This thread's placeholder (Section 5) is deliberately minimal — player name, exam track, sign-out button only |
| User said "don't use this" about an uploaded `index.html` | No such upload or instruction occurred in this thread |

If the receiving agent is continuing from the parallel session rather than this one, treat that session's file (`JBR-Auth-Dashboard.jsx`) and this session's file (`App.jsx`) as **two independent prototypes of the same visual concept** — not two versions of the same file — until reconciled by a human.

---

## 10. NOT YET DONE / NEXT STEPS FOR THE RECEIVING AGENT

0. **Confirm the real-app situation first (see Section 9)** — ask the user whether an existing `index.html` PWA with working Firebase auth actually exists. This determines whether steps 1–2 below mean "build a dashboard and wire up Firebase from scratch" or "port this visual flow into an existing app and existing auth."
1. **Wire up the real Dashboard** — replace the placeholder per Section 5.
2. **Replace mock auth with real Firebase** — the `signIn()` mock block in `useAuth` has the exact replacement code commented inline; persistence logic (localStorage/sessionStorage) does not need to change.
3. **Decide what `examTrack` should actually do** — right now it's just passed through and displayed; it isn't gating any content or logic. If NEET/JEE/CET should change what data loads or how the dashboard behaves, that logic needs to be added.
4. **Real player data** — there is no data-fetching hook left in the file (the old `usePlayerData` mock was removed with the dashboard). The new dashboard will need its own.

---

**File:** `/mnt/user-data/outputs/App.jsx` (single-file React artifact, ~720 lines after cleanup)
