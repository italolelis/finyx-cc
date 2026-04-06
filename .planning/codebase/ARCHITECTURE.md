# Architecture

**Analysis Date:** 2026-04-06

## Pattern Overview

**Overall:** Meta-prompting system / Claude Code slash-command plugin

**Key Characteristics:**
- Not a traditional application — IMMO is a collection of Claude Code slash-commands and AI agent definitions that install into `~/.claude/` or `.claude/`
- All "business logic" is encoded in Markdown prompt files (`.md`), not executable code
- The only runtime code is `bin/install.js` — a Node.js installer that copies files into the Claude config directory
- Commands reference shared reference documents and templates using `@path` directives (Claude Code's file-inclusion syntax)
- Three-agent architecture: orchestrating commands delegate sub-tasks to specialized sub-agents

## Layers

**Installer (Entry Point):**
- Purpose: Deploy IMMO files into Claude's config directory at install time
- Location: `bin/install.js`
- Contains: Node.js CLI, file copy logic with path rewriting
- Depends on: `commands/`, `immo/`, `agents/` source directories
- Used by: End users via `npx immo-cc`

**Commands (Slash-Command Layer):**
- Purpose: Top-level user-facing commands invoked in Claude Code as `/immo:[name]`
- Location: `commands/immo/`
- Contains: Prompt `.md` files with `name`, `description`, `allowed-tools` YAML frontmatter and structured `<process>` XML blocks
- Depends on: `immo/references/`, `immo/templates/` via `@path` includes; delegates to `agents/` sub-agents via `Task` tool
- Used by: Claude Code slash-command system

**Reference Knowledge Base:**
- Purpose: Domain knowledge injected into command context at runtime
- Location: `immo/references/`
- Contains: Methodology rules, country-specific tax rules, Erbpacht detection criteria, transport assessment rubrics
- Key files: `immo/references/methodology.md`, `immo/references/germany/tax-rules.md`, `immo/references/erbpacht-detection.md`, `immo/references/transport-assessment.md`
- Depends on: Nothing (pure knowledge documents)
- Used by: Commands via `@~/.claude/immo/references/` includes in `<execution_context>` blocks

**Templates:**
- Purpose: Schema and output scaffolding for generated project files
- Location: `immo/templates/`
- Contains: `config.json` (investor profile schema), `state.md` (analysis state tracker), `location-research.md` (research output template)
- Depends on: Nothing
- Used by: Commands (especially `init`, `scout`) to produce project-local files

**Sub-Agents:**
- Purpose: Specialist AI agents spawned by commands to handle parallelizable sub-tasks
- Location: `agents/`
- Contains: Agent system prompt `.md` files with `name`, `description`, `tools`, `color` YAML frontmatter
- Key files: `agents/immo-analyzer-agent.md`, `agents/immo-location-scout.md`, `agents/immo-reporter-agent.md`
- Depends on: Investor config from project's `.immo/config.json`; reference docs passed in context by orchestrating command
- Used by: Commands that declare `Task` in `allowed-tools`

**Project Data (Runtime, not in repo):**
- Purpose: Per-project state generated when users run commands
- Location: `.immo/` within user's investment project directory
- Contains: `config.json` (investor profile), `STATE.md` (progress tracker), `research/locations/[location].md`, `analysis/[location]/UNITS.md`, `analysis/[location]/RANKED.md`, `analysis/[location]/SHORTLIST.md`, `output/BRIEFING-[DATE].md`
- Depends on: Commands write these; subsequent commands read them
- Used by: All commands after `init`

## Data Flow

**Workflow: Full Analysis Pipeline**

1. User runs `npx immo-cc --global` → `bin/install.js` copies `commands/`, `agents/`, `immo/` into `~/.claude/`
2. User opens Claude Code, runs `/immo:init` → command reads `immo/templates/config.json`, interacts via `AskUserQuestion`, writes `.immo/config.json` and `.immo/STATE.md` to user's project directory
3. User adds property documents to `properties/[location]/` (PDFs, Excel price lists)
4. User runs `/immo:scout [location]` → command loads `immo/references/erbpacht-detection.md` and `immo/references/transport-assessment.md` via `@path`, spawns `immo-location-scout` sub-agent via `Task`, agent performs web searches, writes `.immo/research/locations/[location].md`
5. User runs `/immo:analyze [location]` → command loads `immo/references/methodology.md` and `immo/references/germany/tax-rules.md`, spawns `immo-analyzer-agent` sub-agent, agent reads price list files from `properties/[location]/`, calculates metrics, writes `.immo/analysis/[location]/UNITS.md` and `.immo/analysis/[location]/RANKED.md`
6. User runs `/immo:filter [location]` → reads `.immo/config.json` criteria, reads `RANKED.md`, produces `.immo/analysis/[location]/SHORTLIST.md` and `EXCLUSIONS.md`
7. User runs `/immo:compare` → reads all `SHORTLIST.md` files, produces unified comparison output
8. User runs `/immo:stress-test` → runs scenario analysis on shortlisted units, writes `.immo/analysis/STRESS-TEST.md`
9. User runs `/immo:report` → reads all analysis data, fills `immo/templates/briefing.md`, writes `.immo/output/BRIEFING-[DATE].md`

**State Management:**
- `.immo/STATE.md` is the global progress tracker, updated by each command to reflect current phase
- `.immo/config.json` is the single source of truth for investor parameters; all metric calculations use `investor.marginalRate` and `assumptions.*` from it
- Commands are stateless — they re-read `.immo/config.json` each invocation

## Key Abstractions

**Slash Command Definition:**
- Purpose: A Claude Code slash command is a Markdown file with YAML frontmatter (`name`, `description`, `allowed-tools`) and a structured prompt body
- Examples: `commands/immo/init.md`, `commands/immo/analyze.md`, `commands/immo/scout.md`
- Pattern: Frontmatter declares tool permissions; `<execution_context>` block lists `@path` includes for reference docs; `<process>` block contains numbered phases with Bash code blocks and output format specifications

**Sub-Agent Definition:**
- Purpose: A Claude Code agent is a Markdown system prompt with YAML frontmatter (`name`, `description`, `tools`, `color`)
- Examples: `agents/immo-analyzer-agent.md`, `agents/immo-location-scout.md`, `agents/immo-reporter-agent.md`
- Pattern: Spawned by commands via the `Task` tool; receive structured input, return structured output consumed by orchestrating command

**Investor Config:**
- Purpose: Centralized investor profile used by all analysis commands
- Examples: `immo/templates/config.json` (template), `.immo/config.json` (runtime)
- Pattern: JSON with sections `investor` (tax data), `strategy` (investment horizon, financing), `criteria` (filters), `assumptions` (calculation constants)

**Reference Document:**
- Purpose: Domain knowledge injected into Claude context at command invocation
- Examples: `immo/references/methodology.md`, `immo/references/germany/tax-rules.md`, `immo/references/erbpacht-detection.md`
- Pattern: Pure Markdown knowledge docs; referenced via `@~/.claude/immo/references/` in command `<execution_context>` blocks

## Entry Points

**Package Install Entry Point:**
- Location: `bin/install.js`
- Triggers: `npx immo-cc [options]`
- Responsibilities: Parse `--global`/`--local`/`--uninstall` flags; copy `commands/immo/`, `immo/`, `agents/` into target `.claude/` directory; rewrite embedded `~/.claude/` path references to match install location; write `VERSION` file

**Command Entry Points (post-install):**
- Location: `~/.claude/commands/immo/[name].md` (after install)
- Triggers: User types `/immo:[name]` in Claude Code
- Responsibilities: Each command is an independent workflow; `init` is the mandatory first step; subsequent commands require `.immo/config.json` to exist

## Error Handling

**Strategy:** Commands include `<error_handling>` sections with explicit recovery messages for common failure cases.

**Patterns:**
- Pre-flight Bash checks at Phase 1 of every command: verify `.immo/config.json` exists and location folder exists before proceeding
- Explicit exclusion logic: Erbpacht detection has a defined action matrix (PROCEED / VERIFY / EXCLUDE) encoded in `immo/references/erbpacht-detection.md`
- Missing data fallbacks: Analyzer agent uses conservative estimates and flags values as `[EST]` when source data is incomplete
- User-facing error messages use consistent banner format

## Cross-Cutting Concerns

**Path Rewriting:** `bin/install.js` rewrites `~/.claude/` references in `.md` files to the actual install path, enabling both global (`~/.claude/`) and local (`./.claude/`) installs to work correctly.

**Context Injection:** Commands use Claude Code's `@path` syntax in `<execution_context>` blocks to inject reference documents into LLM context without requiring explicit file reads at runtime.

**Tool Permissions:** Each command's `allowed-tools` YAML array restricts which Claude tools can be used, enforcing separation of concerns (e.g., `status` only needs Read/Bash/Glob; `scout` also needs WebSearch/WebFetch/Task).

**Germany Focus:** Current implementation is Germany-specific. `immo/references/germany/` contains country-specific rules; `init` command exposes Portugal/Spain as "coming soon." The `investor.country` field in config.json is the extension point for future country support.

---

*Architecture analysis: 2026-04-06*
