# Phase 18: Router + Sub-skill Migration - Context

**Gathered:** 2026-04-12
**Status:** Ready for planning
**Mode:** Auto-generated (infrastructure phase — discuss skipped)

<domain>
## Phase Boundary

Insurance skill dispatches to per-type sub-skills via a router. The current monolithic `skills/insurance/SKILL.md` (health insurance only) must become a router that detects insurance type from user input and dispatches to the appropriate sub-skill prompt. The existing health insurance flow migrates to `skills/insurance/sub-skills/health.md` and works identically after migration.

</domain>

<decisions>
## Implementation Decisions

### Claude's Discretion
All implementation choices are at Claude's discretion — pure infrastructure phase. Use ROADMAP phase goal, success criteria, and codebase conventions to guide decisions.

Key constraints from codebase:
- Skills use `SKILL.md` as entry point with YAML frontmatter (`name`, `description`, `allowed-tools`)
- Agents live under `skills/{name}/agents/`
- References live under `skills/{name}/references/`
- Current insurance SKILL.md is a full health insurance advisor (~600 lines)
- Router must preserve the `/finyx:insurance` command name
- Sub-skills are `.md` prompt files loaded by the router based on keyword detection

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `skills/insurance/SKILL.md` — current health insurance advisor (full implementation)
- `skills/insurance/agents/finyx-insurance-calc-agent.md` — GKV/PKV cost calculator
- `skills/insurance/agents/finyx-insurance-research-agent.md` — PKV provider research
- `skills/insurance/references/germany/health-insurance.md` — health insurance reference doc
- `skills/insurance/references/disclaimer.md` — shared legal disclaimer

### Established Patterns
- Each skill has a single `SKILL.md` entry point
- Skills use `${CLAUDE_SKILL_DIR}` for relative references
- Agents are spawned via `Task` tool
- All skills follow the same YAML frontmatter pattern

### Integration Points
- `/finyx:insurance` is the user-facing command — must continue to work
- Agents reference `${CLAUDE_SKILL_DIR}/references/` paths — these must still resolve after migration
- Profile data at `.finyx/profile.json` is read by both the skill and agents

</code_context>

<specifics>
## Specific Ideas

No specific requirements — infrastructure phase. Refer to ROADMAP phase description and success criteria.

</specifics>

<deferred>
## Deferred Ideas

None — infrastructure phase.

</deferred>
