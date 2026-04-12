---
id: SEED-003
status: dormant
planted: 2026-04-12
planted_during: post v1.2 Health Insurance Advisor
trigger_when: next milestone — this is the highest-priority architectural change
scope: Large
---

# SEED-003: Migrate to Claude Code plugin system with marketplace distribution

## Why This Matters

The Claude Code plugin system is **live and production-ready**. Two registries exist: `claude-plugins-official` (Anthropic) and `claudes-kitchen` (community). Finyx currently distributes via npm (`npx finyx-cc`) with a manual `bin/install.js` — this predates the plugin system and is the legacy approach.

Migrating to the plugin system means:
- **Marketplace discovery** — users find Finyx via `/plugin install finyx` instead of knowing about npm
- **Auto-updates** — plugins auto-update; npm requires manual `npx finyx-cc` re-runs
- **Individual skill installation** — users can install just `finyx-tax` or `finyx-insurance` without the full suite
- **Skills auto-trigger** — no need to memorize `/finyx:tax`; Claude detects "help with taxes" and triggers the skill
- **Commands become thin triggers** — skills hold all logic, commands are optional entry points

**Why now:** The plugin system exists TODAY. Every day we stay on npm-only, we miss marketplace exposure and force users through a manual install flow.

## When to Surface

**Trigger:** Next milestone — this should be the priority

This seed should be presented during `/gsd:new-milestone` when:
- Architecture restructuring is planned
- Distribution improvements are discussed
- User acquisition/discoverability is a goal
- Any new skill/command is being added (restructure first, then add)

## Scope Estimate

**Large** — Full milestone. Every command becomes a skill, agents move into skill directories, reference docs split per skill, plugin.json manifest created, submission to Anthropic directory.

## Research Findings (verified 2026-04-12)

### Plugin System Architecture

**Manifest:** `.claude-plugin/plugin.json` at repo root
```json
{
  "name": "finyx",
  "version": "1.0.0",
  "description": "AI-powered personal finance advisor",
  "skills": ["finyx-tax", "finyx-invest", "finyx-insurance", ...],
  "agents": ["finyx-allocation-agent", ...],
  "commands": ["finyx:help", "finyx:status"]
}
```

**Skill format:** `skills/<name>/SKILL.md` with frontmatter:
```yaml
---
name: finyx-tax
description: German and Brazilian investment tax guidance — triggers on tax questions
allowed-tools: [Read, Bash, WebSearch, AskUserQuestion]
---
```

Supporting files in same directory: `agents/`, `references/`, `examples.md`

**Distribution:** GitHub repo → submit to Anthropic via clau.de/plugin-directory-submission
**Installation:** `/plugin install finyx@<marketplace>`
**Auto-updates:** Enabled by default in marketplace plugins

### Proposed Skill Structure

```
finyx/
├── .claude-plugin/
│   └── plugin.json              # marketplace manifest
├── skills/
│   ├── finyx-tax/
│   │   ├── SKILL.md             # full tax advisor logic (was commands/finyx/tax.md)
│   │   ├── agents/
│   │   │   └── finyx-tax-scoring-agent.md
│   │   └── references/
│   │       ├── germany/tax-investment.md
│   │       └── brazil/tax-investment.md
│   ├── finyx-invest/
│   │   ├── SKILL.md
│   │   ├── agents/
│   │   └── references/
│   ├── finyx-insurance/
│   │   ├── SKILL.md
│   │   ├── agents/
│   │   │   ├── finyx-insurance-calc-agent.md
│   │   │   └── finyx-insurance-research-agent.md
│   │   └── references/
│   │       └── germany/health-insurance.md
│   ├── finyx-insights/
│   │   ├── SKILL.md             # orchestrates across other skills' outputs
│   │   ├── agents/
│   │   │   ├── finyx-allocation-agent.md
│   │   │   ├── finyx-tax-scoring-agent.md
│   │   │   └── finyx-projection-agent.md
│   │   └── references/
│   │       └── insights/
│   ├── finyx-broker/
│   ├── finyx-pension/
│   ├── finyx-profile/           # shared profile management
│   └── finyx-realestate/        # legacy real estate suite
├── commands/finyx/              # thin triggers (optional, for /finyx:* syntax)
│   ├── tax.md                   # one-liner: invokes finyx-tax skill
│   └── ...
├── bin/install.js               # legacy npm fallback
└── package.json                 # npm distribution (keeps working)
```

### Key Design Decisions to Make

1. **Shared profile:** How do independent skills find `.finyx/profile.json`? Convention-based (all skills check `./` and `~/`)? Or a `finyx-profile` skill that other skills depend on?

2. **Cross-skill integration:** `finyx-insights` needs outputs from tax, invest, pension, insurance. When installed standalone, it works with profile data only. When other skills are co-installed, it gets richer data. How to detect co-installed skills?

3. **Reference doc sharing:** Tax rules are used by both `finyx-tax` and `finyx-insights`. Duplicate in each skill, or shared `finyx-core` package?

4. **Backward compatibility:** Users who installed via npm need a migration path. `npx finyx-cc` should detect the plugin system and offer to migrate.

5. **Marketplace submission:** Anthropic reviews plugins. Need to ensure all skills meet their quality bar (disclaimers, no harmful advice, etc.).

### Migration Order

1. Create `plugin.json` manifest
2. Restructure `finyx-profile` as the foundation skill (shared profile)
3. Convert `finyx-tax` as pilot (simplest standalone skill)
4. Convert remaining skills
5. `finyx-insights` last (depends on all others)
6. Submit to Anthropic plugin directory
7. Keep npm as fallback

## Breadcrumbs

- `bin/install.js` — current npm installer, would become fallback
- `commands/finyx/*.md` — 17 commands to convert to skills
- `agents/*.md` — 8 agents to redistribute into skill directories
- `finyx/references/` — reference docs to split per skill
- `package.json` — npm manifest, keep for backward compat
- `.planning/PROJECT.md` — architecture section needs update
- `README.md` — installation instructions need dual-path (plugin + npm)
- Existing skills: `fin-advisor`, `fin-tax`, `fin-research`, `fin-budget` at `~/.claude/skills/` — these are early prototypes of the pattern

## Notes

- The `fin-advisor`, `fin-tax`, `fin-research`, `fin-budget` skills already exist locally — these were the original skill prototypes. The migration would replace them with the full finyx command logic.
- Plugin auto-updates mean users get new features without re-running `npx finyx-cc` — major UX win.
- Individual skill installation means a user who only cares about German taxes can install `finyx-tax` alone without the full suite.
- The `finyx-insights` skill is the cross-cutting integration layer — it's the most complex to make standalone because it synthesizes from all domains.
