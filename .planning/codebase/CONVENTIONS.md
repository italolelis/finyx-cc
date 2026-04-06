# Coding Conventions

**Analysis Date:** 2026-04-06

## Overview

This codebase is a **meta-prompting system** — the primary "code" is structured Markdown (Claude command definitions) and one Node.js installer script. Conventions apply to two distinct layers: the JavaScript installer (`bin/install.js`) and the command/agent Markdown files (`commands/immo/*.md`, `agents/*.md`).

---

## Markdown Command Files

### Frontmatter

Every command file uses YAML frontmatter to declare identity and tool permissions:

```yaml
---
name: immo:analyze
description: Analyze all properties in a location - extract units, calculate metrics, rank by yield
allowed-tools:
  - Read
  - Bash
  - Write
  - Glob
  - Grep
  - Task
---
```

- `name`: always `immo:[verb]` format
- `description`: one-sentence imperative describing what the command does
- `allowed-tools`: explicit allowlist — never use `*` or implicit permissions

### Structure Sections

Commands use consistent XML-like section tags:

| Section | Required | Purpose |
|---------|----------|---------|
| `<objective>` | Yes | What the command does and what it creates |
| `<execution_context>` | Conditional | `@` references to load shared files |
| `<process>` | Yes | Numbered phases with bash snippets |
| `<error_handling>` | Conditional | User-facing error messages |
| `<notes>` | Optional | Edge cases and tips |
| `<scenarios>` | Optional | Variant invocations |

Example from `commands/immo/analyze.md`:
```xml
<objective>
Analyze all properties in a location folder:
1. Extract units from price lists (Excel/PDF)
...
</objective>

<execution_context>
@~/.claude/immo/references/methodology.md
@~/.claude/immo/references/germany/tax-rules.md
</execution_context>
```

### Process Phase Naming

Phases inside `<process>` follow the pattern:

```markdown
## Phase 1: Validation
## Phase 2: Document Discovery
## Phase 3: Development Research
```

- Always numbered with `## Phase N: [Noun]`
- Sub-phases use `### N.N [Noun]` (e.g., `### 4.1 Tax Class`)
- Bash code blocks always show the exact commands to run
- Output banners use `━━━` separator lines with ` IMMO ► COMMAND ` header

### Output Banners

Consistent terminal output format across all commands:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 IMMO ► ANALYZE: [LOCATION]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Placeholder Convention

Template placeholders in output examples use `[UPPER_SNAKE_CASE]`:
- `[LOCATION]`, `[DATE]`, `[COUNT]`, `[PRICE_LIST_FILE]`

Config template placeholders use `{{CAMEL_CASE}}`:
- `{{INVESTMENT_DESCRIPTION}}`, `{{UNITS_TABLE}}`

These are never mixed within the same file.

### File Path References

Output file paths always use the `.immo/` prefix for runtime data:
- `.immo/config.json` — investor profile
- `.immo/analysis/[location]/UNITS.md` — per-location analysis
- `.immo/research/locations/[location].md` — location research
- `.immo/output/BRIEFING-[DATE].md` — reports

Input source files live in:
- `properties/[location]/` — raw developer documents

---

## Agent Files

Agent files (`agents/*.md`) use a different structure — no YAML frontmatter, instead:

```markdown
# IMMO [Role] Agent

## Your Role
## [Process Sections]
## Output Format
## Key Rules
## Error Handling
```

Agent return format is a structured text block (not YAML):

```
LOCATION_RESEARCH_COMPLETE
location: [name]
status: [PROCEED | CAUTION | EXCLUDE]
erbpacht: [NO | SUSPECTED | CONFIRMED]
```

---

## JavaScript (bin/install.js)

### Style

- CommonJS (`require`/`module.exports`), not ESM
- No external dependencies — only Node.js built-ins (`fs`, `path`, `os`)
- Node >= 16.7.0 minimum (see `package.json` `engines`)

### Naming

| Element | Pattern | Example |
|---------|---------|---------|
| Functions | camelCase verbs | `install()`, `uninstall()`, `copyWithPathReplacement()` |
| Variables | camelCase nouns | `targetDir`, `locationLabel`, `pathPrefix` |
| Constants | camelCase | `hasGlobal`, `hasLocal`, `hasUninstall` |
| ANSI colors | single descriptive name | `cyan`, `green`, `yellow`, `dim`, `reset` |

### Function Design

Functions are small and single-purpose:
- `getClaudeDir()` — resolve config path
- `copyWithPathReplacement(srcDir, destDir, pathPrefix)` — recursive copy with regex substitution
- `install(isGlobal)` — orchestrate installation
- `uninstall(isGlobal)` — orchestrate removal

No classes. No async. Synchronous fs operations throughout.

### Error Handling

Errors use `console.error()` + `process.exit(1)`:

```js
if (hasGlobal && hasLocal) {
  console.error(`  ${yellow}Cannot specify both --global and --local${reset}`);
  process.exit(1);
}
```

User-facing messages are indented with 2 spaces and use ANSI color codes.

### CLI Argument Parsing

Manual `process.argv.slice(2)` with `Array.includes()` — no CLI library:

```js
const hasGlobal = args.includes('--global') || args.includes('-g');
const hasLocal  = args.includes('--local')  || args.includes('-l');
```

---

## Data Format Conventions

### config.json Schema

The investor config follows a flat schema with four top-level keys:

```json
{
  "$schema": "https://immo.dev/schemas/config.schema.json",
  "version": "1.0.0",
  "project": { ... },
  "investor": { ... },
  "strategy": { ... },
  "criteria": { ... },
  "assumptions": { ... }
}
```

- All monetary values are plain numbers (not strings)
- Rates are decimal percentages (`44.31` not `0.4431`)
- Dates are ISO 8601 strings (`"2024-01-15T10:00:00Z"`)
- Boolean flags named as `[verb][Noun]` — `parkingRequired`, `excludeErbpacht`

### Markdown Output Tables

All output tables use standard GitHub-flavored Markdown pipe syntax. Column order is always: identifier first, then size/price, then yield, then cashflow, then projections.

### Financial Formatting

- Prices: `€272,403` (English locale, comma separator)
- Portuguese translation: `€272.403`
- German translation: `272.403 €`
- Percentages: two decimal places (`3.16%`)
- Large numbers: rounded to thousands in summaries (`€31k`)

---

## Reference Files

Shared knowledge lives in `immo/references/`:
- `methodology.md` — 10 mandatory rules, ranking criteria, exclusion criteria
- `germany/tax-rules.md` — German-specific tax calculations
- `erbpacht-detection.md` — ground lease detection logic
- `transport-assessment.md` — transport quality scoring

Templates live in `immo/templates/`:
- `config.json` — default investor profile structure
- `briefing.md` — full advisor report template with `{{PLACEHOLDER}}` syntax
- `location-research.md` — location research output template
- `state.md` — STATE.md template

Commands reference these via `@` imports in `<execution_context>`.

---

## Error Message Style

User-facing errors follow a consistent pattern:
- Quoted command suggestions: `Use \`/immo:status\``
- Actionable resolution always provided
- Soft errors (no-op conditions) use yellow color, not red
- Hard exits use `process.exit(1)` in JS; in commands, explicit "Abort" instruction

---

*Convention analysis: 2026-04-06*
