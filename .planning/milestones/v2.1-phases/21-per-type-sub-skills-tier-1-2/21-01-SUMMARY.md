---
phase: 21-per-type-sub-skills-tier-1-2
plan: "01"
subsystem: insurance-sub-skills
tags: [insurance, haftpflicht, hausrat, rechtsschutz, sub-skills, advisory]
dependency_graph:
  requires:
    - "18-router-sub-skill-migration (router SKILL.md)"
    - "19-reference-docs-profile-schema-agents (reference docs + research agent)"
    - "20-portfolio-analysis (portfolio sub-skill pattern)"
  provides:
    - "Privathaftpflicht advisory sub-skill (haftpflicht.md)"
    - "Hausrat advisory sub-skill (hausrat.md)"
    - "Rechtsschutz advisory sub-skill (rechtsschutz.md)"
  affects:
    - "skills/insurance/SKILL.md (router dispatches to these files)"
    - "finyx-insurance-research-agent (spawned by all 3 sub-skills)"
tech_stack:
  added: []
  patterns:
    - "7-phase sub-skill structure: preferences > validation > disclaimer > benchmark > cancellation > research agent > recommendation"
    - "Sum-based benchmark (haftpflicht: Deckungssumme >=5M EUR)"
    - "Computed benchmark (hausrat: living_area_sqm x 650 from reference doc)"
    - "Module-based comparison (rechtsschutz: PASS/MISSING/N/A per life situation)"
    - "Cancellation deadline computation with 30-day ALERT banner and Sonderkündigungsrecht detection"
    - "Research agent spawn via Task with insurance_type slug"
key_files:
  created:
    - skills/insurance/sub-skills/haftpflicht.md
    - skills/insurance/sub-skills/hausrat.md
    - skills/insurance/sub-skills/rechtsschutz.md
  modified: []
decisions:
  - "Removed --- horizontal separators to satisfy automated verification (grep -q '^---' tests for YAML frontmatter absence)"
  - "Hausrat living_area_sqm uses 3-source priority: Phase 0 user input > profile field lookup > UNKNOWN"
  - "Rechtsschutz coverage_amount is null — Phase 3 uses module-based PASS/MISSING/N/A, never monetary benchmark"
  - "N/A vs MISSING distinction in Rechtsschutz: N/A = not relevant for life situation; MISSING = relevant but absent"
metrics:
  duration_seconds: 321
  completed_date: "2026-04-12"
  tasks_completed: 3
  tasks_total: 3
  files_created: 3
  files_modified: 0
---

# Phase 21 Plan 01: Per-Type Sub-skills (Tier 1-2) Summary

**One-liner:** Three insurance sub-skills created — haftpflicht (sum-based ≥€5M Deckungssumme), hausrat (computed living-area benchmark €650/m²), rechtsschutz (module-based PASS/MISSING/N/A per employment/housing/driving situation).

## What Was Built

### Task 1 — haftpflicht.md (Privathaftpflicht sub-skill)

Created `skills/insurance/sub-skills/haftpflicht.md` (~310 lines). Key capabilities:
- Phase 0 collects budget range, renting status, and pet ownership
- Phase 3 reads benchmarks from `${CLAUDE_SKILL_DIR}/references/germany/haftpflicht.md` at runtime — PASS/FAIL on Deckungssumme ≥€5M, Mietsachschaden, Schlüsselverlust, Gefälligkeitsschäden, Forderungsausfalldeckung
- Phase 4 computes cancellation deadline with 30-day ALERT banner and Sonderkündigungsrecht detection
- Phase 5 spawns finyx-insurance-research-agent with `insurance_type: haftpflicht` and user preferences
- Phase 6 flags Hundehaftpflicht gap for dog owners (Privathaftpflicht does NOT cover dog liability)

### Task 2 — hausrat.md (Hausrat sub-skill)

Created `skills/insurance/sub-skills/hausrat.md` (~331 lines). Key capabilities:
- Phase 0 collects living area in m², Elementarschäden interest, Fahrrad-Zusatz interest
- Living area resolution: Phase 0 input → profile field fallback → UNKNOWN (never skips Phase 3)
- Phase 3 computes `minimum_sum = living_area_sqm × 650` (value read from reference doc, not hardcoded)
- Unterversicherungsverzicht FAIL triggers prominent warning about pro-rata claim reduction
- Phase 4 includes Umzug/address-change Sonderkündigungsrecht note
- Phase 5 spawns finyx-insurance-research-agent with `insurance_type: hausrat`

### Task 3 — rechtsschutz.md (Rechtsschutz sub-skill)

Created `skills/insurance/sub-skills/rechtsschutz.md` (~334 lines). Key capabilities:
- Phase 0 collects employment status, renting status, driving status — determines module relevance
- Phase 3 uses module-based comparison only (coverage_amount is null for service-based type)
- N/A vs MISSING distinction: N/A = module not relevant for life situation; MISSING = relevant but absent
- Wartezeit 3-month status check from start_date with "coverage may be limited" warning when active
- Phase 4 notes that ongoing covered disputes are NOT voided by policy cancellation
- Phase 5 spawns finyx-insurance-research-agent with `insurance_type: rechtsschutz` and module relevance context

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] --- horizontal separators caused verification failure**
- **Found during:** Task 1 verification
- **Issue:** Automated verify script used `! grep -q "^---"` to check for absent YAML frontmatter, but `---` horizontal rules in the body also matched
- **Fix:** Replaced all `---` horizontal separators with blank lines using `sed -i`
- **Files modified:** skills/insurance/sub-skills/haftpflicht.md
- **Note:** Tasks 2 and 3 were written without `---` separators from the start to avoid the same issue

## Known Stubs

None — all three sub-skills read benchmarks from reference docs at runtime, spawn a live research agent for market comparison, and compute all values from profile data. No hardcoded thresholds, no placeholder text.

## Self-Check: PASSED

- FOUND: skills/insurance/sub-skills/haftpflicht.md
- FOUND: skills/insurance/sub-skills/hausrat.md
- FOUND: skills/insurance/sub-skills/rechtsschutz.md
- FOUND: .planning/phases/21-per-type-sub-skills-tier-1-2/21-01-SUMMARY.md
- FOUND: commit dbbc130 (haftpflicht)
- FOUND: commit c8ed083 (hausrat)
- FOUND: commit c69cc43 (rechtsschutz)
