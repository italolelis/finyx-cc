---
phase: 20-portfolio-analysis
plan: "02"
subsystem: insights
tags: [insurance, insights, schema-migration, policies-array]
dependency_graph:
  requires: [19-01, 19-02, 19-03]
  provides: [OPT-04]
  affects: [skills/insights/SKILL.md]
tech_stack:
  added: []
  patterns: [insurance.policies[] array read, set-difference gap detection]
key_files:
  modified:
    - skills/insights/SKILL.md
decisions:
  - "Insurance monthly_cost in the allocation agent prohibition text is intentional — the 'Do NOT reference insurance.monthly_cost' instruction is correct and necessary for the agent to avoid using defunct fields"
metrics:
  duration: "~3 minutes"
  completed: "2026-04-12T20:28:24Z"
  tasks_completed: 1
  tasks_total: 1
  files_modified: 1
---

# Phase 20 Plan 02: Insights Insurance Schema Migration Summary

**One-liner:** Migrated insights SKILL.md from defunct `insurance.type`/`monthly_cost` flat fields to `insurance.policies[]` array with premium summing, essential-type gap detection, and updated CAL-05 trigger.

## What Was Done

Updated `skills/insights/SKILL.md` in 4 targeted locations to align with the Phase 19 `insurance.policies[]` schema:

1. **Phase 1 optional fields block** — replaced flat field references with `insurance.policies[]` array read instructions including set-difference gap detection against essential types (haftpflicht, hausrat, health)
2. **Phase 3 allocation agent prompt** — replaced `insurance.type`/`monthly_cost`/`employer_share` logic with `policies[].premium_monthly` summing, `[DATA GAP]` handling for missing premiums, and missing-essential-type recommendations
3. **Phase 5 CAL-05 pattern** — updated trigger from `insurance.type == "PKV"` to checking for a `health` type entry in `insurance.policies[]` with `premium_monthly > 400`
4. **Notes Cross-Skill Data Dependencies** — updated to reference `insurance.policies[]` array

## Verification Results

| Check | Result |
|-------|--------|
| `insurance.policies` appears >= 4 times | PASS (5 occurrences) |
| `insurance.monthly_cost` absent as data source | PASS (1 occurrence in prohibition instruction only) |
| `insurance.employer_share` fully removed | PASS (0 occurrences) |
| `premium_monthly` referenced | PASS (5 occurrences) |
| `Missing insurance` gap action present | PASS |

## Deviations from Plan

**None** — plan executed exactly as written.

**Note on verification:** The automated verify command `! grep -q "insurance.monthly_cost"` technically fails because the replacement text includes the instruction "Do NOT reference insurance.type or insurance.monthly_cost — those fields do not exist in the schema". This is correct content per the plan's Edit 2 specification. The old field is not used as a data source anywhere — the single occurrence is a prohibition notice to the allocation agent. All acceptance criteria are met.

## Commits

| Task | Commit | Description |
|------|--------|-------------|
| Task 1: Update insights SKILL.md | `5f7ea50` | feat(20-02): update insights SKILL.md to read insurance from policies[] array |

## Known Stubs

None — all insurance integration is fully specified in the SKILL.md prompt content.

## Self-Check: PASSED

- [x] `skills/insights/SKILL.md` exists and modified
- [x] Commit `5f7ea50` exists in git log
- [x] `insurance.policies[]` appears 5 times in SKILL.md
- [x] `insurance.employer_share` not present
- [x] `premium_monthly` present
- [x] `Missing insurance` gap action present
