# Research Summary — v1.1 Financial Insights Dashboard

**Researched:** 2026-04-06
**Confidence:** HIGH

## Executive Summary

Finyx v1.1 adds `/fin:insights` — a CLI command that aggregates existing advisor data from `.finyx/profile.json` into a unified financial health report. Zero new dependencies; all logic in Markdown prompts. Follows the proven orchestrator-agent pattern: one command spawns three parallel specialist agents (allocation, tax scoring, projection), which return structured outputs synthesized into a ranked report.

Critical design decision: **profile-only data sourcing**. Reads `.finyx/profile.json` directly, never invokes other advisor commands. Emits coverage gaps for incomplete sections.

## Key Findings

### Stack
- No new dependencies — zero-dependency constraint preserved
- Unicode block characters for CLI visualization
- Standard FV formulas computed inline by Claude
- Two new reference docs: `benchmarks.md` and `scoring-rules.md`

### Features

**P1 (v1.1):** Income allocation vs country-adjusted benchmarks; per-country tax efficiency gaps in €; net worth snapshot; emergency fund check; goal pace indicator; top-5 impact-ranked recommendations; cross-advisor intelligence.

**Anti-features:** Single combined financial health score (gamification trap); budget/expense tracking (different product); combined cross-jurisdiction tax score (misleading).

### Architecture
- 3 parallel agents (allocation, tax-scoring, projection) — no inter-agent dependencies
- All agents read profile.json directly via orchestrator context
- Synthesis (merge, sort by € impact, cross-domain links) in orchestrator only
- Purely additive — no existing files modified

**Build order:** benchmarks.md → scoring-rules.md → 3 agents (parallel) → insights.md orchestrator → bin/install.js update

### Top Pitfalls
1. **Projection precision theater** — always ranges + explicit assumptions, round to €1k
2. **Stale/incomplete data** — completeness gate mandatory before analysis
3. **Cross-jurisdiction score conflation** — per-country scores only, never combined
4. **Gross-income benchmarks** — must compute on net-of-mandatory-deductions income for DE
5. **Disclaimer regression** — disclaimer.md in execution_context + inline qualifiers

## Suggested Phases (3)

1. **Reference Foundation** — benchmarks.md + scoring-rules.md (unblocks all agents)
2. **Specialist Agents** — 3 agents built in parallel (allocation, tax-scoring, projection)
3. **Orchestrator Command** — Wire agents, data-completeness gate, synthesis, disclaimers, installer update

---
*Research completed: 2026-04-06*
