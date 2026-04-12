# Phase 20: Portfolio Analysis - Context

**Gathered:** 2026-04-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Users can see their full insurance portfolio with cost summary, gaps, overlaps, and adequacy assessment. The portfolio sub-skill is triggered via the router when the user says "portfolio", "overview", or "summary". It spawns the portfolio agent and synthesizes results into conversational advisory output. Insurance total cost feeds into /finyx:insights allocation section.

</domain>

<decisions>
## Implementation Decisions

### Portfolio Output Structure
- Portfolio overview organized by tier classification (Mandatory > Essential > Recommended > Situational) matching reference doc tiers
- Gap detection automatically considers family situation from profile.json (family_status, children)
- Overlap detection presented as warning banners per overlap with affected policies named inline
- Adequacy benchmarks show threshold source (e.g., "Hausrat: €650/m² per GDV recommendation")

### Insights Integration
- Insurance appears as a new "Insurance" row in /finyx:insights allocation table with monthly total + % of net income
- Insights shows insurance gaps as action items in recommendations section ("Missing: Haftpflicht, Rechtsschutz")
- Portfolio sub-skill triggered by "portfolio"/"overview"/"summary" keyword in /finyx:insurance args

### Sub-skill Architecture
- Portfolio sub-skill spawns the portfolio agent via Task tool (consistent with health sub-skill pattern)
- When no policies exist in profile, prompt user to enter policies interactively via AskUserQuestion before analysis
- Output is conversational only — no file writes (matches health sub-skill pattern)

### Claude's Discretion
- Exact table formatting and column choices for portfolio overview
- How to present the total monthly cost summary
- Specific wording of gap/overlap/adequacy warnings
- How interactive policy entry collects data (number of AskUserQuestion rounds)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- skills/insurance/SKILL.md — router with keyword map (portfolio keywords need adding)
- skills/insurance/agents/finyx-insurance-portfolio-agent.md — portfolio agent (from Phase 19)
- skills/insurance/references/germany/*.md — 11 reference docs with tier classifications and benchmarks
- skills/profile/references/profile.json — insurance.policies[] schema
- skills/insights/SKILL.md — insights skill that needs insurance allocation row

### Established Patterns
- Sub-skills are plain Markdown at skills/insurance/sub-skills/{type}.md
- Agents spawned via Task tool with structured input/output XML blocks
- All advisory output preceded by disclaimer
- Conversational output, no file writes

### Integration Points
- Router SKILL.md needs "portfolio"/"overview"/"summary" keywords added
- Portfolio sub-skill spawns portfolio agent
- /finyx:insights needs to read insurance.policies[] for cost allocation
- Profile.json insurance.policies[] is the data source

</code_context>

<specifics>
## Specific Ideas

No specific requirements beyond the decisions above.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>
