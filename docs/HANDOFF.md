# Session Handoff — 2026-04-28

## Repos in scope
- `lychan110/contact-scout` — React + Vite, GitHub Pages deployment
- `lychan110/rsvp-automation` — Vanilla JS HTML apps, GitHub Pages on `master`

---

## What was done this session

### 1. CLAUDE.md improvements (both repos, merged)
Both repos received a new **UI/UX Development Approach** section:
- Agile iteration rules (one interaction per commit, promote to prod file on approval)
- UX principles table (empty states, inline feedback, destructive confirmations, outcome labels, visible progress)
- Responsive layout guidelines (max-width containers, overflow-x scroll on tables, test at 1440/900 px)
- Dark theme consistency rules (reuse existing colors, require :hover/:focus states)

PRs merged: `contact-scout#1`, `rsvp-automation#18`

### 2. contact-scout: empty deployed site + accessibility audit (merged)
**Root cause of empty site:** `.github/workflows/static.yml` was uploading the raw repo
(including uncompiled `.tsx` source) instead of building first.

**Fixes applied** (all in `contact-scout` PR #2, merged to `main`):

| File | Change |
|------|---------|
| `.github/workflows/static.yml` | Added `npm ci` + `npm run build` steps; changed upload path from `'.'` to `'./dist'` |
| `vite.config.ts` | Added `base: './'` so asset URLs are relative and resolve under GitHub Pages subpath |
| `index.html` + `App.tsx` | `height: 100vh` → `height: 100dvh` (mobile browser toolbar fix) |
| `src/App.tsx` | `#6e7681` → `#8b949e` everywhere (contrast 4.36:1 → 6.42:1, WCAG AA pass) |
| `src/App.tsx` | `#2d3340` → `#7d8590` for old log entries (was 1.88:1, now 5.86:1) |
| `src/App.tsx` | Hint text `#21262d` → `#8b949e` (was invisible, 1.33:1) |
| `src/App.tsx` | Status badge font 8px → 10px |
| `src/App.tsx` | Removed `outline: none` from inputs; added `*:focus-visible` ring |

**Deployed at:** `https://lychan110.github.io/contact-scout/`  
Password: `scout2025`

---

## Current state of both repos

### contact-scout
- `main` is clean and fully deployed
- No open PRs

### rsvp-automation
- `master` is clean (merged CLAUDE.md improvements)
- No open PRs
- No build step — raw HTML files, deployed as-is

---

## Outstanding / next steps

### contact-scout
- [ ] **Customize scan prompts** — `src/App.tsx` `SCAN_PROMPTS` still has placeholders (`[YOUR STATE]`, `[YOUR COUNTIES]`, `[CITY 1-3]`). The app shows a warning banner until these are replaced.
- [ ] **Phone preview** — tunnel services are blocked by the environment's network allowlist. Test on phone via the live GitHub Pages URL instead.
- [ ] **Model version** — currently hardcoded to `claude-sonnet-4-20250514`. Update if a newer Sonnet model is released.
- [ ] **Responsive audit** — fixed two-column grid (`1fr 220px`) stacks awkwardly at narrow viewports. Consider collapsing the log panel on mobile.

### rsvp-automation
- [ ] Review `docs/ROADMAP.md`, `docs/PROGRESS.md`, `docs/TASKS.md` for queued UX work
- [ ] Keep planning docs in sync with CLAUDE.md when doing UI work

---

## Key conventions

**contact-scout (React + Vite):**
- Inline styles throughout — no CSS modules or Tailwind
- Dark GitHub-style palette; primary accent `#1f6feb`, text `#f0f6fc`, muted `#8b949e`
- `saveState()` via `useEffect` on every state change
- API key in `sessionStorage` only (never localStorage), key `cs_api_key`
- Model: `claude-sonnet-4-20250514` with `web_search_20250305` tool
- Dev: `npm run dev` on port 5174; build: `npm run build` → `dist/`

**rsvp-automation (vanilla JS):**
- Single `<script>` block per file, state at top, `render()` on every mutation
- No frameworks, no imports, no fetch except Claude API + Google APIs
- Google OAuth via GSI CDN — never hardcode API keys
- Tab index changes must update TABS array, render() switch, onclick, and render checks together
- Production files: `inviteflow.html`, `contactscout.html` — version files use `_v{N:02d}.html`

---

## Environment notes
- Node 22, npm 10 at `/opt/node22`
- Network allowlist blocks tunnel services — phone preview must use the live GitHub Pages URL
- `obra/episodic-memory` cannot be installed in the sandbox (native `better-sqlite3` addon fails); install locally with `npm install -g github:obra/episodic-memory` and register with `claude mcp add episodic-memory episodic-memory-mcp-server`
