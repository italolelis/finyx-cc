---
phase: 22-per-type-sub-skills-tier-3-4-doc-reader
plan: "01"
subsystem: insurance-skill
tags: [insurance, reise, fahrrad, sub-skills, profile-schema]
requirements: [TYPE-07, TYPE-08]

dependency_graph:
  requires:
    - 19-01 (reference docs: reise.md, fahrrad.md)
    - 18-01 (router SKILL.md — loads sub-skills at dispatch time)
  provides:
    - skills/insurance/sub-skills/reise.md
    - skills/insurance/sub-skills/fahrrad.md
    - skills/profile/references/profile.json (documents.locations schema)
  affects:
    - finyx-insurance-research-agent (spawned by both sub-skills)
    - doc-reader sub-skill (documents.locations schema foundation)

tech_stack:
  added: []
  patterns:
    - "Service-based sub-skill with component presence table (reise — no sum benchmark)"
    - "Sum-based sub-skill with Neuwert benchmark comparison (fahrrad)"
    - "Conditional branch logic in Phase 3 (e-bike rows) and Phase 4 (Einmalreise skip)"
    - "Unconditional educational warning pattern (GKV gap — always shown)"
    - "Overlap detection pattern (Hausrat Fahrrad-Zusatz)"

key_files:
  created:
    - skills/insurance/sub-skills/reise.md
    - skills/insurance/sub-skills/fahrrad.md
  modified:
    - skills/profile/references/profile.json

decisions:
  - "Reise Phase 3 uses component presence table (not sum benchmark) — coverage_amount is null per service_based coverage_type annotation in reise.md reference doc"
  - "GKV gap warning is unconditional in both Phase 3 and Phase 6 — educational content, not conditional on missing coverage"
  - "Einmalreise cancellation skip triggered by Phase 0 selection OR notes/coverage_components text match — dual detection"
  - "Fahrrad e-bike branch adds Akkuschaden + Elektronikschaden rows only when is_ebike == Yes — avoids false FAILs for standard bikes"
  - "Hausrat overlap warning emitted when has_hausrat_fahrrad == Yes from Phase 0 — prompts comparison before double-insurance"
  - "documents.locations inserted after insurance section and before project section in profile.json — logical grouping with schema comment"

metrics:
  duration_minutes: 4
  completed_date: "2026-04-12"
  tasks_completed: 2
  tasks_total: 2
  files_created: 2
  files_modified: 1
---

# Phase 22 Plan 01: Reise + Fahrrad Sub-skills + Profile Documents Schema Summary

**One-liner:** Travel and bicycle insurance sub-skills with service/sum-based benchmarks, GKV gap warning, e-bike branch, and Hausrat overlap detection; profile schema extended with documents.locations for doc-reader.

## What Was Built

### Task 1: Profile schema extension + Reiseversicherung sub-skill

**skills/insurance/sub-skills/reise.md** — 7-phase travel insurance advisory following the established haftpflicht.md pattern with key differences:

- Phase 0 collects 3 preferences: policy_scope (Jahreskarte/Einmalreise/Not sure), travel_region, budget_range
- Phase 3 is service-based: no sum benchmark, evaluates component presence (Auslandskrankenversicherung, Krankenrücktransport, Reiserücktritt, Reisegepäck, Reiseabbruch, per-trip duration limit)
- GKV gap warning emitted unconditionally before the comparison table — not conditional on missing coverage
- Phase 4 skips cancellation tracking when policy_scope == "Einmalreise" from Phase 0 OR when notes/coverage_components contain "Einmalreise" — single-trip policies are non-cancellable by design
- Phase 5 spawns finyx-insurance-research-agent with insurance_type: reise and all Phase 0 preferences
- Phase 6 always leads with GKV adequacy statement; includes Jahreskarte vs Einmalreise cost-effectiveness guidance

**skills/profile/references/profile.json** — Extended with two new sections after insurance and before project:
- `_documents_schema`: schema comment block documenting path_format and categories
- `documents.locations`: { insurance: null, banking: null, investments: null, real_estate: null } — foundation for future doc-reader sub-skill

### Task 2: Fahrradversicherung sub-skill

**skills/insurance/sub-skills/fahrrad.md** — 7-phase bicycle insurance advisory:

- Phase 0 collects 4 preferences: bike_value_eur (text input), is_ebike, has_hausrat_fahrrad, budget_range
- Phase 3 is sum-based: Versicherungssumme compared against bike_value_eur (Neuwert from Phase 0)
- E-bike branch: if is_ebike == Yes, adds Akkuschaden and Elektronikschaden rows as Required — with note that battery replacement costs EUR 500–2,000
- Hausrat overlap detection: if has_hausrat_fahrrad == Yes, emits overlap warning banner prompting comparison of Hausrat add-on limit against Neuwert before purchasing standalone coverage
- Phase 5 spawns finyx-insurance-research-agent with insurance_type: fahrrad and all Phase 0 preferences including bike_value_eur and is_ebike
- Phase 6 leads with Hausrat overlap analysis if detected; flags Akkuschaden gap as CRITICAL when e-bike without battery coverage; includes Neuwert adequacy assessment

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None — all sub-skills read benchmarks from reference docs at runtime via the Read tool. No hardcoded values.

## Self-Check: PASSED

- `skills/insurance/sub-skills/reise.md` exists: CONFIRMED
- `skills/insurance/sub-skills/fahrrad.md` exists: CONFIRMED
- `skills/profile/references/profile.json` valid JSON with documents.locations: CONFIRMED
- Commits exist:
  - 3f0627f feat(22-01): add Reiseversicherung sub-skill and documents schema extension
  - c62b7d8 feat(22-01): add Fahrradversicherung sub-skill with e-bike branch and Hausrat overlap detection
