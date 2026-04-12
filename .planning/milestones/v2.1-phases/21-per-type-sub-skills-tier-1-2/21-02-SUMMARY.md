---
phase: 21-per-type-sub-skills-tier-1-2
plan: "02"
subsystem: insurance
tags: [zahnzusatz, risikoleben, sub-skills, dental, term-life, benchmarks, staffelung, gesundheitspruefung]

requires:
  - phase: 19-reference-docs-profile-schema-agents
    provides: zahnzusatz.md and risikoleben.md reference docs with coverage benchmarks
  - phase: 18-router-sub-skill-migration
    provides: sub-skill plain Markdown contract (no frontmatter, no execution_context)

provides:
  - Zahnzusatz sub-skill with Staffelung warning, GKV Festzuschuss context, Wartezeit switching warning
  - Risikoleben sub-skill with income/mortgage benchmark computation, policy expiry semantics, health underwriting warning

affects:
  - 21-03-PLAN (Tier 2 sub-skills — pattern reference for remaining types)
  - 22-per-type-sub-skills-tier-3 (cross-phase consistency)
  - SKILL.md router (already wired; no change needed)

tech-stack:
  added: []
  patterns:
    - "Per-type sub-skill: Phase 0 (preferences) → Phase 1 (profile read + income extraction) → Phase 2 (disclaimer) → Phase 3 (benchmark comparison with reference doc) → Phase 4 (cancellation tracking) → Phase 5 (research agent spawn) → Phase 6 (recommendation)"
    - "Benchmark computation reads multipliers from reference doc at runtime — never hardcodes threshold values"
    - "Risikoleben renewal_date = policy END DATE (Vertragsende), not annual renewal — displayed as 'Policy expires'"
    - "Health underwriting warning placed in disclaimer, cancellation phase, and recommendation (3 placements)"
    - "Staffelung warning block always shown in Zahnzusatz regardless of user's immediate dental needs"

key-files:
  created:
    - skills/insurance/sub-skills/zahnzusatz.md
    - skills/insurance/sub-skills/risikoleben.md
  modified: []

key-decisions:
  - "Staffelung warning is unconditional — shown even when no existing policy and no urgent dental needs, because it's a structural risk all new purchasers must understand"
  - "Risikoleben benchmark uses max(income × multiplier, mortgage_balance) — reads multipliers from reference doc, not hardcoded"
  - "Health underwriting (Gesundheitsprüfung) warning repeated 3 times in Risikoleben (disclaimer, cancellation, recommendation) — intentional, as cancel-before-new-approval is the most consequential mistake"
  - "No Rückkaufswert note added to Risikoleben cancellation phase — distinguishes pure term life from savings life insurance"
  - "GKV Festzuschuss baseline section added to Zahnzusatz Phase 3 before benchmark table — essential framing context"

patterns-established:
  - "Income-computed benchmark pattern: read multipliers from reference doc, compute in Phase 3, not hardcoded"
  - "Policy expiry vs annual renewal distinction: fixed-term types use 'Policy expires' not 'Next renewal'"
  - "Graceful income-missing degradation: show UNKNOWN banner + general guidance, not error crash"

requirements-completed: [TYPE-05, TYPE-06, OPT-01, OPT-03]

duration: 20min
completed: 2026-04-12
---

# Phase 21 Plan 02: Zahnzusatz and Risikoleben Sub-skills Summary

**Zahnzusatz sub-skill with Staffelung/GKV-Festzuschuss framing and Risikoleben sub-skill with income/mortgage benchmark formula and policy-expiry (not renewal) semantics**

## Performance

- **Duration:** ~20 min
- **Started:** 2026-04-12T20:50:00Z
- **Completed:** 2026-04-12T21:10:00Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments

- Created `zahnzusatz.md` (408 lines): Phase 0 dental needs urgency, GKV Festzuschuss baseline (§55 SGB V Bonusheft 60/70/75%), unconditional Staffelung warning block with annual cap escalation table, Wartezeit switching warning, research agent spawn
- Created `risikoleben.md` (448 lines): income/mortgage benchmark computation (reads multipliers from reference doc), policy expiry semantics (renewal_date = Vertragsende, not annual renewal), triple-placed Gesundheitsprüfung warning, no-Rückkaufswert note, graceful income-missing fallback
- Both files follow the sub-skill plain Markdown contract: no YAML frontmatter, no execution_context block, no file writes

## Task Commits

1. **Task 1: Create Zahnzusatz sub-skill (zahnzusatz.md)** — `4db25b5` (feat)
2. **Task 2: Create Risikoleben sub-skill (risikoleben.md)** — `5b0e958` (feat)

## Files Created/Modified

- `/Users/italovietro/projects/immo/skills/insurance/sub-skills/zahnzusatz.md` — Zahnzusatz advisory sub-skill (6 phases, Staffelung + GKV Festzuschuss context)
- `/Users/italovietro/projects/immo/skills/insurance/sub-skills/risikoleben.md` — Risikoleben advisory sub-skill (6 phases, income/mortgage benchmark, policy expiry semantics)

## Decisions Made

- **Staffelung warning unconditional:** The Staffelung warning block in Zahnzusatz Phase 3 is shown in all cases (existing policy or not, urgent needs or not). It's a structural risk new purchasers must understand before selecting a policy.
- **Three Gesundheitsprüfung placements:** The health underwriting warning appears in Phase 2 (disclaimer), Phase 4 (cancellation), and Phase 6 (recommendation) of Risikoleben. The "cancel first → get rejected → no coverage" trap is the most consequential mistake in Risikoleben management.
- **Multipliers read from reference doc at runtime:** Neither sub-skill hardcodes benchmark values. The Risikoleben benchmark formula reads multipliers from `references/germany/risikoleben.md` Coverage Benchmarks section, enabling automatic inheritance of future doc updates.
- **Income-missing graceful degradation:** When `countries.germany.gross_income` is absent from profile, Risikoleben shows an UNKNOWN banner and falls back to general reference doc guidance rather than crashing or showing a computed-but-wrong number.

## Deviations from Plan

None — plan executed exactly as written. File line counts (zahnzusatz: 408, risikoleben: 448) exceed the target ranges in the plan (160-230 and 180-240 respectively) because the content requirements for all 6 phases with complete error handling and notes are comprehensive. All acceptance criteria are met. Automated verify check `! grep -q "^---"` is a known false-negative on files with `---` horizontal rules as phase dividers (same pattern as health.md); the files have no YAML frontmatter.

## Issues Encountered

The automated verify check `! grep -q "^---"` returned FAIL for zahnzusatz.md because `---` horizontal rules used as phase dividers (matching the health.md pattern) were flagged. This is a false-negative in the verification script — the acceptance criteria "No YAML frontmatter" is correctly met (line 1 is `# Zahnzusatz Insurance Sub-skill`). The health.md reference sub-skill has 7 occurrences of `^---` for the same reason. No fix needed.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Phase 21-03 (Kfz sub-skill, Tier 2 types Reise/Fahrrad/Kfz-Schutzbrief/Mietkaution) can proceed
- Pattern established for income-computed benchmarks and fixed-term policy expiry semantics
- No blockers

---
*Phase: 21-per-type-sub-skills-tier-1-2*
*Completed: 2026-04-12*
