# Codebase Structure

**Analysis Date:** 2026-04-06

## Directory Layout

```
immo/                           # npm package root
├── bin/
│   └── install.js              # CLI entry point (npx immo-cc)
├── commands/
│   └── immo/                   # Slash-command prompt files
│       ├── init.md             # /immo:init
│       ├── scout.md            # /immo:scout
│       ├── analyze.md          # /immo:analyze
│       ├── filter.md           # /immo:filter
│       ├── compare.md          # /immo:compare
│       ├── stress-test.md      # /immo:stress-test
│       ├── rates.md            # /immo:rates
│       ├── report.md           # /immo:report
│       ├── status.md           # /immo:status
│       ├── update.md           # /immo:update
│       └── help.md             # /immo:help
├── immo/                       # Reference knowledge + templates
│   ├── references/
│   │   ├── methodology.md      # Core analysis rules
│   │   ├── erbpacht-detection.md
│   │   ├── transport-assessment.md
│   │   └── germany/
│   │       └── tax-rules.md    # German tax calculation rules
│   ├── templates/
│   │   ├── config.json         # Investor config schema/default
│   │   ├── state.md            # STATE.md scaffold
│   │   └── location-research.md
│   └── workflows/              # (empty, reserved)
├── agents/
│   ├── immo-analyzer-agent.md  # Property metrics sub-agent
│   ├── immo-location-scout.md  # Location research sub-agent
│   └── immo-reporter-agent.md  # Report generation sub-agent
├── hooks/                      # (empty, reserved)
├── scripts/                    # (empty, reserved)
├── .github/
│   └── workflows/              # CI/CD
├── package.json
├── README.md
└── LICENSE
```

## Directory Purposes

**`bin/`:**
- Purpose: npm binary entry point
- Contains: `install.js` — the only executable Node.js file in the project
- Key files: `bin/install.js`

**`commands/immo/`:**
- Purpose: Slash-command definitions installed to `~/.claude/commands/immo/`
- Contains: One `.md` file per `/immo:[name]` command; each file is a self-contained Claude prompt with YAML frontmatter and structured XML body
- Key files: `commands/immo/init.md`, `commands/immo/scout.md`, `commands/immo/analyze.md`, `commands/immo/report.md`

**`immo/references/`:**
- Purpose: Domain knowledge injected into command context via `@path` includes
- Contains: Methodology rules, Erbpacht detection guide, transport rubric, country-specific tax tables
- Key files: `immo/references/methodology.md`, `immo/references/germany/tax-rules.md`, `immo/references/erbpacht-detection.md`, `immo/references/transport-assessment.md`

**`immo/templates/`:**
- Purpose: Scaffolding files that commands use to produce project-local output
- Contains: `config.json` (investor profile template with all fields and defaults), `state.md` (STATE.md scaffold), `location-research.md` (research output template)
- Key files: `immo/templates/config.json`, `immo/templates/state.md`

**`agents/`:**
- Purpose: Claude sub-agent definitions; spawned by commands via the `Task` tool
- Contains: One `.md` file per agent with YAML frontmatter (`name`, `description`, `tools`, `color`) and agent system prompt
- Key files: `agents/immo-analyzer-agent.md`, `agents/immo-location-scout.md`, `agents/immo-reporter-agent.md`

**`immo/workflows/`:**
- Purpose: Reserved directory (empty)
- Contains: Nothing currently

**`hooks/`:**
- Purpose: Reserved directory (empty)
- Contains: Nothing currently

**`scripts/`:**
- Purpose: Reserved directory (empty)
- Contains: Nothing currently

## Key File Locations

**Entry Points:**
- `bin/install.js`: npm binary; handles `--global`, `--local`, `--uninstall` flags; copies package contents into Claude config dir

**Configuration:**
- `package.json`: npm metadata; `bin` points to `bin/install.js`; `files` array lists what gets published (`bin`, `commands`, `immo`, `agents`, `scripts`)
- `immo/templates/config.json`: Canonical schema for `.immo/config.json`; contains all valid fields with example values

**Core Prompt Logic:**
- `commands/immo/init.md`: Mandatory first command; creates project structure and `.immo/config.json`
- `commands/immo/scout.md`: Location research orchestrator; spawns `immo-location-scout` agent
- `commands/immo/analyze.md`: Metrics calculation orchestrator; spawns `immo-analyzer-agent`
- `commands/immo/report.md`: Report generation; supports `--short`, `--pdf`, `--lang` flags

**Domain Knowledge:**
- `immo/references/methodology.md`: Defines the 5 core analysis principles and ranked metric priorities
- `immo/references/germany/tax-rules.md`: German Sonder-AfA, Spekulationsfrist, marginal rate tables
- `immo/references/erbpacht-detection.md`: Erbpacht detection rules and action matrix
- `immo/references/transport-assessment.md`: Transport quality rubric for parking necessity decisions

**Sub-Agents:**
- `agents/immo-analyzer-agent.md`: Handles price list extraction and full metric calculations (yield, cashflow, tax benefit, 10-year exit)
- `agents/immo-location-scout.md`: Handles web research, Erbpacht detection, transport assessment, market conditions
- `agents/immo-reporter-agent.md`: Handles briefing document generation from analysis data

## Naming Conventions

**Files:**
- Slash-command files: lowercase, hyphenated, `.md` extension matching the command name (`stress-test.md` → `/immo:stress-test`)
- Agent files: `immo-[role].md` prefix, all lowercase hyphenated (`immo-analyzer-agent.md`)
- Reference files: lowercase hyphenated noun phrases (`erbpacht-detection.md`, `tax-rules.md`)
- Template files: lowercase noun (`config.json`, `state.md`)

**Directories:**
- Source directories: lowercase, no hyphens (`commands`, `agents`, `immo`, `bin`)
- Sub-namespace: `commands/immo/` — the `immo` subdirectory creates the slash-command namespace `/immo:`
- Country-specific: nested under locale code (`references/germany/`)

**Runtime Output (user project, not in repo):**
- State file: `.immo/STATE.md` (uppercase, single file)
- Research outputs: `.immo/research/locations/[location].md` (lowercase location name)
- Analysis outputs: `.immo/analysis/[location]/UNITS.md`, `RANKED.md`, `SHORTLIST.md`, `EXCLUSIONS.md` (uppercase, location-namespaced)
- Reports: `.immo/output/BRIEFING-[DATE].md`, `.immo/output/SUMMARY-[DATE].md` (uppercase with date suffix)
- Mortgage research: `.immo/research/market/RATES-[DATE].md`

## Where to Add New Code

**New slash-command:**
- Implementation: `commands/immo/[name].md`
- Pattern: Copy frontmatter structure from existing command (e.g., `commands/immo/status.md`); declare `allowed-tools` array; add `<execution_context>` with `@path` references to needed docs; write `<process>` phases

**New country support:**
- Tax rules: `immo/references/[country]/tax-rules.md`
- Update: `commands/immo/init.md` Phase 2 country selection options
- Update: `commands/immo/analyze.md` to branch on `investor.country`

**New sub-agent:**
- Agent definition: `agents/immo-[role].md`
- Register: Reference in the orchestrating command via `Task` tool call

**New reference document:**
- Location: `immo/references/[name].md`
- Register: Add `@~/.claude/immo/references/[name].md` to the `<execution_context>` block of commands that need it

**New template:**
- Location: `immo/templates/[name].[ext]`
- Register: Reference from command that uses it in `<execution_context>` block

## Special Directories

**`.immo/` (runtime, in user project):**
- Purpose: All generated project state and analysis output
- Generated: Yes, by IMMO commands
- Committed: User's choice; should be committed to track analysis progress

**`.planning/` (this repo):**
- Purpose: GSD planning documents
- Generated: Yes, by GSD commands
- Committed: Yes

**`immo/workflows/`:**
- Purpose: Reserved for future workflow definitions
- Generated: No
- Committed: Yes (empty directory)

---

*Structure analysis: 2026-04-06*
