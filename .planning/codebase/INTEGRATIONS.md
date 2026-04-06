# External Integrations

**Analysis Date:** 2026-04-06

## Overview

This is a Claude Code plugin — all external integrations are performed by Claude at runtime using Claude Code's built-in tools (`WebSearch`, `WebFetch`, `Bash`). There are no SDK clients, API keys, or hardcoded credentials in the codebase.

## Web Search & Fetch (Claude Code Tools)

**Used by:** `commands/immo/scout.md`, `commands/immo/rates.md`, `commands/immo/report.md`, `agents/immo-location-scout.md`

**Tools:** `WebSearch`, `WebFetch` — declared in command frontmatter `allowed-tools`

**Searches performed at runtime:**
- German mortgage rate sites: `drklein.de`, `baufi24.de`, `interhyp.de`, `finanztip.de` (see `commands/immo/rates.md`)
- German real estate portals: `immobilienscout24.de`, `immowelt.de` (see `agents/immo-location-scout.md`)
- General web search for location/development research (Neubau, Erbpacht, transport)

**No API keys required** — Claude Code's WebSearch/WebFetch tools handle authentication.

## GitHub API

**Used by:** `commands/immo/update.md`

**Endpoint:** `https://api.github.com/repos/italolelis/immo/releases/latest`

**Purpose:** Fetch release notes when updating IMMO to latest version.

**Auth:** Unauthenticated public API call (no token required for public repos).

## npm Registry

**Used by:** `commands/immo/update.md`, `bin/install.js`

**Endpoint:** Default npm registry (`registry.npmjs.org`)

**Purpose:**
- Check latest published version: `npm view immo-cc version`
- Install/update: `npx immo-cc@latest`

**Auth:** OIDC trusted publishing via GitHub Actions — no `_authToken` stored anywhere. See `.github/workflows/publish.yml`.

## PDF Converters (Optional, Runtime)

**Used by:** `commands/immo/report.md` (when `--pdf` flag is passed)

**Tools checked in order at runtime (via `Bash` tool):**
1. `pandoc` + `xelatex` — Preferred, must be installed by user (`brew install pandoc`)
2. `npx md-to-pdf` — Node.js fallback, pulled via npx
3. `mdpdf` — Alternative Node.js fallback

**No integration required** — commands detect and invoke whichever is available.

## Data Storage

**Databases:**
- None. All data is stored as local files in the user's project.

**File Storage:**
- Local filesystem only.
- Project data structure (created in user's project, not in this repo):
  - `.immo/config.json` — Investor profile and criteria
  - `.immo/STATE.md` — Analysis state tracking
  - `.immo/research/locations/[location].md` — Location research
  - `.immo/research/market/RATES-[DATE].md` — Rate research
  - `.immo/analysis/[location]/UNITS.md` — Extracted units
  - `.immo/analysis/[location]/RANKED.md` — Ranked analysis
  - `.immo/analysis/[location]/SHORTLIST.md` — Filtered shortlist
  - `.immo/output/BRIEFING-[DATE].md` — Advisor reports (Markdown)
  - `.immo/output/BRIEFING-[DATE].pdf` — Advisor reports (PDF, if generated)
  - `properties/[location]/` — User-supplied price lists (Excel, PDF)

**Caching:**
- None.

## Authentication & Identity

**Auth Provider:**
- None. No user authentication in IMMO itself.
- Claude Code handles authentication to its own services.

## CI/CD & Deployment

**Hosting:**
- npm registry (`registry.npmjs.org`) — package distribution

**CI Pipeline:**
- GitHub Actions (`.github/workflows/publish.yml`)
- Trigger: push of `v*` tags
- Steps: checkout → Node 22 setup → npm 11.5+ install → version verification → `npm publish --access public` → GitHub Release creation
- Auth: OIDC trusted publishing (`id-token: write` permission, no stored secrets)

## Monitoring & Observability

**Error Tracking:** None.

**Logs:** Console output only (ANSI color codes), written during `bin/install.js` execution.

## Webhooks & Callbacks

**Incoming:** None.

**Outgoing:** None.

## Environment Configuration

**Required env vars:** None — the package runs without any env vars.

**Optional env vars:**
- `CLAUDE_CONFIG_DIR` — Override Claude config directory (default: `~/.claude`). Read in `bin/install.js`.

**Secrets location:** None stored. npm publish uses OIDC; no tokens in repo or environment.

## External Data Sources Referenced in Methodology

These are web sources that Claude searches at runtime (not hardcoded integrations):

| Source | Purpose | Command |
|--------|---------|---------|
| drklein.de | Mortgage rate research | `rates.md` |
| interhyp.de | Mortgage rate research | `rates.md` |
| baufi24.de | Mortgage rate research | `rates.md` |
| finanztip.de | Mortgage rate research | `rates.md` |
| immobilienscout24.de | Market data | `scout.md`, agents |
| immowelt.de | Market data | `scout.md`, agents |
| City statistics offices | Population/demand data | `scout.md` |
| api.github.com | Release notes for update | `update.md` |

---

*Integration audit: 2026-04-06*
