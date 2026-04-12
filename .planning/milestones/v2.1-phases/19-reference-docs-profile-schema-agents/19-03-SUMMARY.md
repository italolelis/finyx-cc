---
phase: 19-reference-docs-profile-schema-agents
plan: "03"
subsystem: insurance-agents
tags: [agents, insurance, portfolio, document-reader, research, generic]
dependency_graph:
  requires: ["19-01", "19-02"]
  provides: ["ARCH-02", "ARCH-03", "ARCH-04"]
  affects: ["phase-21", "phase-22"]
tech_stack:
  added: []
  patterns:
    - "Generic parameterized agent (insurance_type dispatch)"
    - "Coverage type awareness (sum_based vs service_based) in portfolio analysis"
    - "PDF extraction via Read tool with Field Extraction Schema from reference doc"
key_files:
  created:
    - skills/insurance/agents/finyx-insurance-portfolio-agent.md
    - skills/insurance/agents/finyx-insurance-doc-reader-agent.md
  modified:
    - skills/insurance/agents/finyx-insurance-research-agent.md
decisions:
  - "Health type retains existing 3-provider PKV comparison in Phase 4b of research agent; all other types use criteria-only output per §34d GewO"
  - "Portfolio agent loads all 11 reference docs unconditionally in execution_context (minimal overhead, simplifies dispatch logic)"
  - "Doc reader constructs policy id as {type}-{provider_slug}-{year} — deterministic, no UUID dependency"
  - "Exclusions extracted separately from coverage_components — flagged for user review, not written to profile"
metrics:
  duration_seconds: 257
  completed: "2026-04-12"
  tasks_completed: 2
  files_created: 2
  files_modified: 1
---

# Phase 19 Plan 03: Specialist Agents Summary

**One-liner:** Three insurance agents created — generic research agent parameterized by insurance_type with §34d GewO criteria-only output, portfolio gap/overlap analyzer reading insurance.policies[], and PDF document reader using reference doc Field Extraction Schema.

## What Was Built

### Task 1: Generalized Research Agent (ARCH-02)

Rewrote `finyx-insurance-research-agent.md` from PKV-specific to generic:
- Accepts `insurance_type` parameter (any of 11 types)
- Loads matching reference doc via `${CLAUDE_SKILL_DIR}/references/germany/${insurance_type}.md`
- For non-health types: returns criteria-based output only — coverage criteria checklist, benchmarks, red flags, market context. Never returns provider/tariff names (§34d GewO).
- For health type: retains full PKV 3-provider comparison in Phase 4b (health insurance advisory predates constraint formalization)
- All benchmarks sourced from reference doc (anti-hallucination rule)
- Loads `disclaimer.md` in execution_context

### Task 2: Portfolio Agent (ARCH-03) + Document Reader Agent (ARCH-04)

**`finyx-insurance-portfolio-agent.md`** — new agent:
- Reads `insurance.policies[]` from `.finyx/profile.json`
- Loads all 11 reference docs + disclaimer.md in execution_context
- Phase 2: coverage adequacy check with sum_based vs service_based type awareness
- Phase 3: gap detection with MANDATORY/ESSENTIAL/OPTIONAL priority classification using identity fields (family_status, children)
- Phase 4: overlap detection for 3 hardcoded overlap patterns (fahrrad, reise/kfz-schutzbrief, rechtsschutz)
- Returns `<portfolio_analysis>` XML with adequacy table, gaps table, overlaps table, cost summary

**`finyx-insurance-doc-reader-agent.md`** — new agent:
- Accepts `doc_path` + `insurance_type` parameters
- Loads Field Extraction Schema from type-specific reference doc
- Extracts all `insurance.policies[]` fields using German label matching from Keyword Map
- Returns `<doc_reader_result>` with JSON object matching profile schema + per-field confidence flags
- Handles image PDFs, unknown types, and missing files with clear error messages
- Exclusions extracted separately (for user review, not persisted)

## Commits

| Task | Commit | Description |
|------|--------|-------------|
| Task 1 | 6f7787c | feat(19-03): generalize insurance research agent to accept any insurance type |
| Task 2 | 13022d7 | feat(19-03): create portfolio and document reader agents |

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None — all agents are fully specified with complete process phases, output formats, and error handling. No placeholder data or hardcoded empty values that would prevent plan goals from being achieved.

## Self-Check: PASSED
