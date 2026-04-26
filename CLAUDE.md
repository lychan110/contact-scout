# ContactScout — Claude Guide

Author: Lenya Chan
Updated: 2026-04-25

## Purpose

ContactScout is a standalone React + Vite tool for discovering and verifying elected officials
using the Claude API (with web_search tool). It exports invitee lists in InviteFlow's JSON
schema for direct import into the InviteFlow invite automation suite.

This app is Lenya-only. Password gate uses the constant `SCOUT_PW = 'scout2025'`.

## Schema contract with InviteFlow

When the user clicks **Export → InviteFlow JSON**, the downloaded file contains:

```json
{
  "exportedAt": "ISO datetime",
  "source": "ContactScout",
  "count": 42,
  "invitees": [
    {
      "firstName": "",
      "lastName": "",
      "title": "",
      "category": "US Senate|US House|Executive|State Senate|House|City Council",
      "email": "",
      "rsvpLink": "",
      "inviteStatus": "pending",
      "sentAt": "",
      "rsvpStatus": "No Response",
      "rsvpDate": "",
      "notes": ""
    }
  ]
}
```

InviteFlow's `InviteesTab` imports this via **Import JSON** → the `importJSON` function reads
`firstName`, `lastName`, `title`, `category`, `email` from each record.

Rules:
- Officials with `status === 'left_office'` are excluded from export.
- Only officials with a non-empty email are included.
- `notes` = `"district · county"` where available.
- `inviteStatus` defaults to `"pending"`.

## Architecture

Single React app with Vite. No router. State lives in the top-level `App` component.

```
src/
  main.tsx         — mounts <App />, applies global styles
  App.tsx          — full ContactScout logic (password gate + inner panel)
  types.ts         — CSOfficial, CSScanMeta, Invitee types
index.html
vite.config.ts
package.json
```

## Persistence

- `localStorage` key: `contactscout_state` — stores `{ officials, newOfficials, scanStatus, scanMeta }`
- `sessionStorage` key: `cs_api_key` — stores Claude API key for this browser session
- `sessionStorage` key: `cs_unlocked` — set to `'1'` after password gate is passed

## Claude API key setup

ContactScout calls the Anthropic API directly from the browser. No backend required.

1. Go to https://console.anthropic.com/ → API Keys → Create Key.
2. Copy the key (starts with `sk-ant-`).
3. In the app, click **⚙ Key** in the header and paste the key.
4. The key is stored in `sessionStorage` only — it is never persisted to `localStorage`
   or sent anywhere except `https://api.anthropic.com/v1/messages`.

Required header: `anthropic-dangerous-direct-browser-access: true` (Anthropic allowlist for
browser-direct usage).

Model used: `claude-sonnet-4-20250514` with the `web_search_20250305` tool.

## Customization before first use

The scan prompts in `src/App.tsx` (`SCAN_PROMPTS`) contain placeholder text:
`[YOUR STATE]`, `[YOUR COUNTIES]`, `[CITY 1]`, `[CITY 2]`, `[CITY 3]`.

Replace these with the actual state, counties, and cities before scanning.
`needsCustomization()` detects these placeholders and shows a banner in the Scan tab.

## Dev setup

```bash
npm install
npm run dev      # http://localhost:5174
npm run build    # dist/
```

## Scan targets

Each of the 7 scan targets calls Claude with a web-search-enabled prompt:
- US Congress (all seats for your state)
- State Executive Branch
- State Senate (all seats)
- State House (tracked counties)
- City Council ×3

After scanning, new officials surface as add candidates. Accepted candidates move to the
Verify Existing tab. Run re-scans after each election cycle.

## Export formats

| Button | Format | Purpose |
|--------|--------|---------|
| Export → InviteFlow JSON | `.json` with `invitees[]` in InviteFlow schema | Import into InviteFlow |
| ↓ Backup | `.json` with full state | Restore/backup |
| ↓ CSV | `.csv` with all official fields | Spreadsheet analysis |

## Conventions

- No framework other than React + Vite.
- Inline styles throughout (matches InviteFlow's visual language — dark GitHub-style).
- `saveState()` called in a `useEffect` whenever `officials / newOfficials / scanStatus / scanMeta` change.
- Author credit "by Lenya Chan" visible in the welcome banner.
