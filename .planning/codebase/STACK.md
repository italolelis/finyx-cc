# Technology Stack

**Analysis Date:** 2026-04-06

## Languages

**Primary:**
- JavaScript (Node.js) - `bin/install.js` (installer CLI)
- Markdown - All command definitions (`commands/immo/*.md`), agent definitions (`agents/*.md`), reference docs (`immo/references/`)

**Secondary:**
- JSON - Configuration templates (`immo/templates/config.json`), package manifest (`package.json`)

## Runtime

**Environment:**
- Node.js >=16.7.0 (declared in `package.json` `engines` field)
- Node.js 22 used in CI (`.github/workflows/publish.yml`)

**Package Manager:**
- npm (11.5+ required for OIDC publishing per CI workflow)
- No lockfile committed (package has no runtime dependencies)

## Frameworks

**Core:**
- None — This is a Claude Code slash-command plugin distributed via npm. There is no application framework.
- Claude Code SDK (implicit) — Commands are Markdown files with YAML frontmatter that Claude Code interprets as slash commands. Agents use the Claude Code agent system.

**Testing:**
- Not applicable — No test framework detected.

**Build/Dev:**
- None — No build step. Files are installed directly via `bin/install.js`.

## Key Dependencies

**Runtime dependencies:**
- None — `package.json` has no `dependencies` or `devDependencies` fields.

**External CLI tools (invoked at runtime by commands, not packaged):**
- `pandoc` — Optional, for PDF report generation (`commands/immo/report.md`)
- `md-to-pdf` / `mdpdf` — Optional npm tools for PDF fallback (`commands/immo/report.md`)
- `bash` / POSIX shell — Used by commands via Claude Code's `Bash` tool

## Configuration

**Environment:**
- No `.env` file or environment variable loading in the package itself.
- `CLAUDE_CONFIG_DIR` — Optional env var that overrides the default install target (`~/.claude`), read in `bin/install.js`.
- Project-level config lives at `.immo/config.json` in the user's investment project (not in this repo).

**Build:**
- No build config files. Package is published as-is.

## Platform Requirements

**Development:**
- Node.js >=16.7.0
- npm for publishing
- Claude Code installed for using the commands

**Production:**
- Distributed via npm as `immo-cc` (`npm install -g immo-cc` or `npx immo-cc`)
- Installs to `~/.claude/` (global) or `.claude/` (local) within user's project
- Claude Code must be present on the target machine to execute commands

## Package Distribution

**npm package name:** `immo-cc`
**Current version:** `0.1.7` (`package.json`)
**Entry binary:** `bin/install.js`
**Published files:** `bin/`, `commands/`, `immo/`, `agents/`, `scripts/`
**Publish trigger:** Git tags matching `v*` via GitHub Actions (`.github/workflows/publish.yml`)
**npm publish auth:** OIDC trusted publishing (no token stored)

---

*Stack analysis: 2026-04-06*
