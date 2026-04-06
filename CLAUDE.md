<!-- GSD:project-start source:PROJECT.md -->
## Project

**Finyx**

Finyx is an open-source personal finance advisor built as a collection of Claude Code slash-commands and specialist AI agents. It evolves the existing IMMO real estate analysis tool into a comprehensive financial planning system — covering tax optimization, investment portfolio management, insurance comparison, pension planning, and real estate investment. Users interact through a single CLI interface (`/fin:*` commands) while specialist agents handle domain-specific analysis backed by shared user context and memory.

**Core Value:** A single AI-powered financial advisor that knows your full financial picture — tax situation, investments, insurance, pensions, and real estate — and gives integrated, country-aware advice rather than siloed recommendations.

### Constraints

- **Tech stack**: Claude Code slash-command architecture — no application framework, all logic in Markdown prompts
- **Runtime**: Node.js >=16.7.0, npm distribution, zero runtime dependencies
- **Legal**: Advisory only — all recommendations include disclaimers, no automated execution
- **Data freshness**: Market data via live web search + APIs, tax rules in versioned reference docs updated per tax year
- **Country scope**: Germany + Brazil for v1, architecture must support adding countries without refactoring
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- JavaScript (Node.js) - `bin/install.js` (installer CLI)
- Markdown - All command definitions (`commands/immo/*.md`), agent definitions (`agents/*.md`), reference docs (`immo/references/`)
- JSON - Configuration templates (`immo/templates/config.json`), package manifest (`package.json`)
## Runtime
- Node.js >=16.7.0 (declared in `package.json` `engines` field)
- Node.js 22 used in CI (`.github/workflows/publish.yml`)
- npm (11.5+ required for OIDC publishing per CI workflow)
- No lockfile committed (package has no runtime dependencies)
## Frameworks
- None — This is a Claude Code slash-command plugin distributed via npm. There is no application framework.
- Claude Code SDK (implicit) — Commands are Markdown files with YAML frontmatter that Claude Code interprets as slash commands. Agents use the Claude Code agent system.
- Not applicable — No test framework detected.
- None — No build step. Files are installed directly via `bin/install.js`.
## Key Dependencies
- None — `package.json` has no `dependencies` or `devDependencies` fields.
- `pandoc` — Optional, for PDF report generation (`commands/immo/report.md`)
- `md-to-pdf` / `mdpdf` — Optional npm tools for PDF fallback (`commands/immo/report.md`)
- `bash` / POSIX shell — Used by commands via Claude Code's `Bash` tool
## Configuration
- No `.env` file or environment variable loading in the package itself.
- `CLAUDE_CONFIG_DIR` — Optional env var that overrides the default install target (`~/.claude`), read in `bin/install.js`.
- Project-level config lives at `.immo/config.json` in the user's investment project (not in this repo).
- No build config files. Package is published as-is.
## Platform Requirements
- Node.js >=16.7.0
- npm for publishing
- Claude Code installed for using the commands
- Distributed via npm as `immo-cc` (`npm install -g immo-cc` or `npx immo-cc`)
- Installs to `~/.claude/` (global) or `.claude/` (local) within user's project
- Claude Code must be present on the target machine to execute commands
## Package Distribution
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Overview
## Markdown Command Files
### Frontmatter
- `name`: always `immo:[verb]` format
- `description`: one-sentence imperative describing what the command does
- `allowed-tools`: explicit allowlist — never use `*` or implicit permissions
### Structure Sections
| Section | Required | Purpose |
|---------|----------|---------|
| `<objective>` | Yes | What the command does and what it creates |
| `<execution_context>` | Conditional | `@` references to load shared files |
| `<process>` | Yes | Numbered phases with bash snippets |
| `<error_handling>` | Conditional | User-facing error messages |
| `<notes>` | Optional | Edge cases and tips |
| `<scenarios>` | Optional | Variant invocations |
### Process Phase Naming
## Phase 1: Validation
## Phase 2: Document Discovery
## Phase 3: Development Research
- Always numbered with `## Phase N: [Noun]`
- Sub-phases use `### N.N [Noun]` (e.g., `### 4.1 Tax Class`)
- Bash code blocks always show the exact commands to run
- Output banners use `━━━` separator lines with ` IMMO ► COMMAND ` header
### Output Banners
### Placeholder Convention
- `[LOCATION]`, `[DATE]`, `[COUNT]`, `[PRICE_LIST_FILE]`
- `{{INVESTMENT_DESCRIPTION}}`, `{{UNITS_TABLE}}`
### File Path References
- `.immo/config.json` — investor profile
- `.immo/analysis/[location]/UNITS.md` — per-location analysis
- `.immo/research/locations/[location].md` — location research
- `.immo/output/BRIEFING-[DATE].md` — reports
- `properties/[location]/` — raw developer documents
## Agent Files
## Your Role
## [Process Sections]
## Output Format
## Key Rules
## Error Handling
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
- `getClaudeDir()` — resolve config path
- `copyWithPathReplacement(srcDir, destDir, pathPrefix)` — recursive copy with regex substitution
- `install(isGlobal)` — orchestrate installation
- `uninstall(isGlobal)` — orchestrate removal
### Error Handling
### CLI Argument Parsing
## Data Format Conventions
### config.json Schema
- All monetary values are plain numbers (not strings)
- Rates are decimal percentages (`44.31` not `0.4431`)
- Dates are ISO 8601 strings (`"2024-01-15T10:00:00Z"`)
- Boolean flags named as `[verb][Noun]` — `parkingRequired`, `excludeErbpacht`
### Markdown Output Tables
### Financial Formatting
- Prices: `€272,403` (English locale, comma separator)
- Portuguese translation: `€272.403`
- German translation: `272.403 €`
- Percentages: two decimal places (`3.16%`)
- Large numbers: rounded to thousands in summaries (`€31k`)
## Reference Files
- `methodology.md` — 10 mandatory rules, ranking criteria, exclusion criteria
- `germany/tax-rules.md` — German-specific tax calculations
- `erbpacht-detection.md` — ground lease detection logic
- `transport-assessment.md` — transport quality scoring
- `config.json` — default investor profile structure
- `briefing.md` — full advisor report template with `{{PLACEHOLDER}}` syntax
- `location-research.md` — location research output template
- `state.md` — STATE.md template
## Error Message Style
- Quoted command suggestions: `Use \`/immo:status\``
- Actionable resolution always provided
- Soft errors (no-op conditions) use yellow color, not red
- Hard exits use `process.exit(1)` in JS; in commands, explicit "Abort" instruction
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Pattern Overview
- Not a traditional application — IMMO is a collection of Claude Code slash-commands and AI agent definitions that install into `~/.claude/` or `.claude/`
- All "business logic" is encoded in Markdown prompt files (`.md`), not executable code
- The only runtime code is `bin/install.js` — a Node.js installer that copies files into the Claude config directory
- Commands reference shared reference documents and templates using `@path` directives (Claude Code's file-inclusion syntax)
- Three-agent architecture: orchestrating commands delegate sub-tasks to specialized sub-agents
## Layers
- Purpose: Deploy IMMO files into Claude's config directory at install time
- Location: `bin/install.js`
- Contains: Node.js CLI, file copy logic with path rewriting
- Depends on: `commands/`, `immo/`, `agents/` source directories
- Used by: End users via `npx immo-cc`
- Purpose: Top-level user-facing commands invoked in Claude Code as `/immo:[name]`
- Location: `commands/immo/`
- Contains: Prompt `.md` files with `name`, `description`, `allowed-tools` YAML frontmatter and structured `<process>` XML blocks
- Depends on: `immo/references/`, `immo/templates/` via `@path` includes; delegates to `agents/` sub-agents via `Task` tool
- Used by: Claude Code slash-command system
- Purpose: Domain knowledge injected into command context at runtime
- Location: `immo/references/`
- Contains: Methodology rules, country-specific tax rules, Erbpacht detection criteria, transport assessment rubrics
- Key files: `immo/references/methodology.md`, `immo/references/germany/tax-rules.md`, `immo/references/erbpacht-detection.md`, `immo/references/transport-assessment.md`
- Depends on: Nothing (pure knowledge documents)
- Used by: Commands via `@~/.claude/immo/references/` includes in `<execution_context>` blocks
- Purpose: Schema and output scaffolding for generated project files
- Location: `immo/templates/`
- Contains: `config.json` (investor profile schema), `state.md` (analysis state tracker), `location-research.md` (research output template)
- Depends on: Nothing
- Used by: Commands (especially `init`, `scout`) to produce project-local files
- Purpose: Specialist AI agents spawned by commands to handle parallelizable sub-tasks
- Location: `agents/`
- Contains: Agent system prompt `.md` files with `name`, `description`, `tools`, `color` YAML frontmatter
- Key files: `agents/immo-analyzer-agent.md`, `agents/immo-location-scout.md`, `agents/immo-reporter-agent.md`
- Depends on: Investor config from project's `.immo/config.json`; reference docs passed in context by orchestrating command
- Used by: Commands that declare `Task` in `allowed-tools`
- Purpose: Per-project state generated when users run commands
- Location: `.immo/` within user's investment project directory
- Contains: `config.json` (investor profile), `STATE.md` (progress tracker), `research/locations/[location].md`, `analysis/[location]/UNITS.md`, `analysis/[location]/RANKED.md`, `analysis/[location]/SHORTLIST.md`, `output/BRIEFING-[DATE].md`
- Depends on: Commands write these; subsequent commands read them
- Used by: All commands after `init`
## Data Flow
- `.immo/STATE.md` is the global progress tracker, updated by each command to reflect current phase
- `.immo/config.json` is the single source of truth for investor parameters; all metric calculations use `investor.marginalRate` and `assumptions.*` from it
- Commands are stateless — they re-read `.immo/config.json` each invocation
## Key Abstractions
- Purpose: A Claude Code slash command is a Markdown file with YAML frontmatter (`name`, `description`, `allowed-tools`) and a structured prompt body
- Examples: `commands/immo/init.md`, `commands/immo/analyze.md`, `commands/immo/scout.md`
- Pattern: Frontmatter declares tool permissions; `<execution_context>` block lists `@path` includes for reference docs; `<process>` block contains numbered phases with Bash code blocks and output format specifications
- Purpose: A Claude Code agent is a Markdown system prompt with YAML frontmatter (`name`, `description`, `tools`, `color`)
- Examples: `agents/immo-analyzer-agent.md`, `agents/immo-location-scout.md`, `agents/immo-reporter-agent.md`
- Pattern: Spawned by commands via the `Task` tool; receive structured input, return structured output consumed by orchestrating command
- Purpose: Centralized investor profile used by all analysis commands
- Examples: `immo/templates/config.json` (template), `.immo/config.json` (runtime)
- Pattern: JSON with sections `investor` (tax data), `strategy` (investment horizon, financing), `criteria` (filters), `assumptions` (calculation constants)
- Purpose: Domain knowledge injected into Claude context at command invocation
- Examples: `immo/references/methodology.md`, `immo/references/germany/tax-rules.md`, `immo/references/erbpacht-detection.md`
- Pattern: Pure Markdown knowledge docs; referenced via `@~/.claude/immo/references/` in command `<execution_context>` blocks
## Entry Points
- Location: `bin/install.js`
- Triggers: `npx immo-cc [options]`
- Responsibilities: Parse `--global`/`--local`/`--uninstall` flags; copy `commands/immo/`, `immo/`, `agents/` into target `.claude/` directory; rewrite embedded `~/.claude/` path references to match install location; write `VERSION` file
- Location: `~/.claude/commands/immo/[name].md` (after install)
- Triggers: User types `/immo:[name]` in Claude Code
- Responsibilities: Each command is an independent workflow; `init` is the mandatory first step; subsequent commands require `.immo/config.json` to exist
## Error Handling
- Pre-flight Bash checks at Phase 1 of every command: verify `.immo/config.json` exists and location folder exists before proceeding
- Explicit exclusion logic: Erbpacht detection has a defined action matrix (PROCEED / VERIFY / EXCLUDE) encoded in `immo/references/erbpacht-detection.md`
- Missing data fallbacks: Analyzer agent uses conservative estimates and flags values as `[EST]` when source data is incomplete
- User-facing error messages use consistent banner format
## Cross-Cutting Concerns
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
