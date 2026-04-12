# Phase 21: Per-Type Sub-skills (Tier 1-2) - Context

**Gathered:** 2026-04-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Create 6 per-type insurance sub-skills for high-priority types: Privathaftpflicht, Hausrat, Kfz, Rechtsschutz, Zahnzusatz, and Risikoleben. Each sub-skill provides detailed analysis, coverage-vs-benchmark comparison, criteria-based market comparison via research agent, and cancellation deadline tracking. Router already has keywords for all types (added in Phase 19).

</domain>

<decisions>
## Implementation Decisions

### Sub-skill Output Format
- Each sub-skill follows the health sub-skill's multi-phase pattern (preferences > validation > agent spawn > disclaimer > synthesis > recommendation) for consistency
- "Current coverage vs benchmark" presented as side-by-side table: Your Coverage vs Recommended Minimum, with pass/fail per criterion
- Market comparison spawns the research agent via Task tool (consistent with health pattern, enables live WebSearch)
- Sub-skills are read-only advisory — no file writes (consistent with health sub-skill pattern)

### Kfz Complexity
- 3 coverage levels presented as tabbed comparison: Haftpflicht vs Teilkasko vs Vollkasko with cost delta and when-to-upgrade guidance
- SF-Klasse (no-claims bonus) collected via AskUserQuestion (0-35 range) with explanation
- Kfz sub-skill has extended Phase with vehicle-specific questions (car age, annual km, garage) before agent spawn

### Cancellation Tracking
- Cancellation deadline data read from profile insurance.policies[] (kuendigungsfrist_months, vertrag_beginn, sonderkundigungsrecht fields)
- Sonderkuendigungsrecht windows presented as alert banner when open or within 30 days ("You have X days to cancel without penalty")
- When cancellation fields are missing from profile: show "Unknown — add policy details via /finyx:insurance portfolio" with link

### Claude's Discretion
- Exact question sets for each sub-skill's preferences phase (varies by type complexity)
- How many research agent search queries per type
- Specific wording of benchmark comparison tables
- How to present the recommendation (prose vs structured)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- skills/insurance/sub-skills/health.md — template pattern for all sub-skills (~600 lines)
- skills/insurance/sub-skills/portfolio.md — second sub-skill example (~274 lines)
- skills/insurance/agents/finyx-insurance-research-agent.md — generic research agent (parameterized by type)
- skills/insurance/references/germany/*.md — 11 reference docs with benchmarks and extraction schemas
- skills/insurance/SKILL.md — router with all 11 type keywords already wired

### Established Patterns
- Sub-skills: plain Markdown, no frontmatter, no execution_context, starts with "# [Type] Insurance Sub-skill"
- Agent spawn via Task tool with structured input/output XML blocks
- Disclaimer emitted before advisory content
- Reference docs loaded via CLAUDE_SKILL_DIR/references/germany/{type}.md

### Integration Points
- Router SKILL.md already has keywords for all 6 types
- Research agent accepts insurance_type parameter
- Profile insurance.policies[] has cancellation fields
- Reference docs have coverage benchmarks per type

</code_context>

<specifics>
## Specific Ideas

No specific requirements beyond the decisions above.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>
