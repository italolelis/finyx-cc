# Research Summary — v1.2 Health Insurance Advisor

**Researched:** 2026-04-08
**Confidence:** HIGH

## Executive Summary

Finyx v1.2 adds `/finyx:insurance` — a PKV vs GKV decision advisor for Germany. No API exists for PKV tariffs; provider research uses WebSearch. GKV is fully computable from statutory constants. The critical design decisions: health data is ephemeral only (GDPR Art. 9), two agents (calculation + research), and four distinct calculation paths (single employee, family, self-employed, Beamter).

## Key Findings

### Stack
- No new dependencies. GKV formula is pure arithmetic from statutory constants. PKV estimates from web search.
- New reference doc: `finyx/references/germany/health-insurance.md` — encodes 2026 constants (JAEG €77,400, BBG €62,100, base rate 14.6%, PV rates, employer caps)
- PKV provider data via WebSearch — no API, Check24/krankenkasseninfo.de as sources
- Long-term projection: two competing curves computed inline

### Features

**Table stakes:** Eligibility check (Versicherungspflichtgrenze gate), GKV cost calculation, PKV cost estimate with health surcharge, family impact (Familienversicherung), tax deduction netting, age-55 lock-in warning.

**Differentiators:** Health questionnaire with risk tier scoring, long-term 10/20/30-year projection with scenarios, Beamter/Beihilfe redirect, expat Anwartschaft guidance, Beitragsrückerstattung and Selbstbeteiligung modeling, cross-advisor integration with /finyx:tax and /finyx:insights.

**Anti-features:** Binding quote generation, storing health diagnoses, recommending specific providers as "best."

### Architecture
- Two agents: `finyx-insurance-calc-agent.md` (deterministic, Read/Grep/Glob) + `finyx-insurance-research-agent.md` (WebSearch/WebFetch for live tariffs)
- Health data: session-only, never persisted. Only non-sensitive fields (employment type, family count) stored in `.finyx/insurance-config.json` on user consent.
- Build order: reference doc → both agents (parallel) → command → profile integration

### Top Pitfalls
1. **Family cost underestimation** — GKV Familienversicherung is free; PKV charges per person
2. **Age-55 lock-in** — §6 Abs. 3a SGB V blocks GKV return permanently
3. **JAEG ≠ BBG confusion** — Eligibility threshold vs contribution ceiling are different numbers
4. **Beamte need separate model** — Beihilfe covers 50-80%, standard PKV model overstates cost
5. **Health data privacy** — GDPR Art. 9 special category, session-only mandatory
6. **Stale tariff data** — PKV premiums change annually, all figures must be live-searched
7. **Linear projection fallacy** — PKV grew 10-20% in 2026 due to Krankenhausreform; use scenarios not single lines

## Suggested Phases (4)

1. **Reference Foundation** — `health-insurance.md` with 2026 constants, formulas, 4 calculation paths
2. **Specialist Agents** — Calc agent (deterministic) + Research agent (WebSearch) in parallel
3. **Command + Questionnaire** — `/finyx:insurance` with eligibility gate, health questionnaire, cost comparison, projection, disclaimers
4. **Cross-Advisor Integration** — Feed insurance costs to `/finyx:insights` and `/finyx:tax`, profile schema extension

---
*Research completed: 2026-04-08*
