# Phase 19: Reference Docs + Profile Schema + Agents - Context

**Gathered:** 2026-04-12
**Status:** Ready for planning
**Mode:** Auto-generated (infrastructure phase — discuss skipped)

<domain>
## Phase Boundary

All supporting infrastructure for per-type insurance advisory: 11 per-type reference docs with coverage benchmarks and legal minimums, profile schema extension for insurance.policies[] array, generic research agent parameterized by insurance type, portfolio agent for cross-type analysis, and document reader agent for PDF parsing. All outputs must include the section 34d GewO advisory-only disclaimer and use criteria-based language (never specific tariff names).

</domain>

<decisions>
## Implementation Decisions

### Claude's Discretion
All implementation choices are at Claude's discretion — pure infrastructure phase. Use ROADMAP phase goal, success criteria, and codebase conventions to guide decisions.

Key constraints from requirements:
- ARCH-02: Generic research agent parameterized by insurance type
- ARCH-03: Portfolio agent for cross-type analysis (gaps, overlaps, total cost)
- ARCH-04: Document reader agent for PDF policy parsing
- ARCH-05: 11 per-type reference docs with coverage benchmarks, legal minimums, field extraction schemas
- INFRA-01: Profile schema extended with insurance.policies[] array
- INFRA-02: Legal disclaimer includes section 34d GewO advisory-only notice
- INFRA-03: All output uses criteria-based recommendations, never specific product recommendations

Reference doc types needed (11 total):
1. Privathaftpflicht (personal liability)
2. Hausratversicherung (household contents)
3. Kfz-Versicherung (car insurance)
4. Rechtsschutzversicherung (legal protection)
5. Zahnzusatzversicherung (dental supplement)
6. Risikolebensversicherung (term life)
7. Reiseversicherung (travel)
8. Fahrradversicherung (bicycle)
9. Kfz-Schutzbriefversicherung (roadside assistance)
10. Mietkautionsversicherung (rental deposit)
11. Krankenversicherung (health — already exists, may need schema alignment)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- skills/insurance/references/germany/health-insurance.md — existing reference doc pattern to follow
- skills/insurance/references/disclaimer.md — existing disclaimer (may need section 34d GewO addition)
- skills/insurance/agents/finyx-insurance-calc-agent.md — existing agent pattern
- skills/insurance/agents/finyx-insurance-research-agent.md — existing agent pattern
- skills/insurance/SKILL.md — router (from Phase 18) that dispatches to sub-skills
- .finyx/profile.json schema — needs insurance.policies[] extension

### Established Patterns
- Reference docs use Markdown with YAML frontmatter (tax_year, etc.)
- Agents use YAML frontmatter with name, description, tools, color fields
- Profile is JSON at .finyx/profile.json

### Integration Points
- Reference docs will be loaded by sub-skills via CLAUDE_SKILL_DIR/references/germany/
- Agents will be spawned by sub-skills via Task tool
- Profile schema is read by all skills

</code_context>

<specifics>
## Specific Ideas

No specific requirements — infrastructure phase. Refer to ROADMAP phase description and success criteria.

</specifics>

<deferred>
## Deferred Ideas

None — infrastructure phase.

</deferred>
