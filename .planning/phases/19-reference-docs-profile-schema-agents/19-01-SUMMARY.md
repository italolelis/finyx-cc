---
phase: 19-reference-docs-profile-schema-agents
plan: "01"
subsystem: insurance
tags: [insurance, profile-schema, disclaimer, reference-docs, legal]
dependency_graph:
  requires: []
  provides:
    - skills/insurance/references/disclaimer.md (section 34d GewO notice)
    - skills/profile/references/profile.json (insurance.policies[] schema)
    - skills/insurance/SKILL.md (11-type keyword map)
    - skills/insurance/references/germany/health-insurance.md (Field Extraction Schema)
  affects:
    - All insurance sub-skills that load disclaimer.md via execution_context
    - Phase 19 Plan 02 (reference docs use profile.json field names)
    - Phase 19 Plan 03 (agents use disclaimer.md and profile.json schema)
tech_stack:
  added: []
  patterns:
    - _insurance_schema doc block in profile.json (matches _holdings_schema pattern)
    - Field Extraction Schema section in reference docs (new pattern for document reader agent)
key_files:
  created: []
  modified:
    - skills/insurance/references/disclaimer.md
    - skills/profile/references/profile.json
    - skills/insurance/SKILL.md
    - skills/insurance/references/germany/health-insurance.md
decisions:
  - "Use short slugs (hausrat, haftpflicht, kfz) not full German names as type enum — matches reference doc filenames and sub-skill filenames"
  - "premium_monthly and premium_annual as separate fields — annual-pay discounts are common"
  - "coverage_amount nullable — service-based types (rechtsschutz, reise) do not have a fixed sum"
  - "Field Extraction Schema in health-insurance.md added additively — existing sections 1-6 unchanged"
metrics:
  duration: "10 minutes"
  completed: "2026-04-12T19:49:00Z"
  tasks_completed: 2
  tasks_total: 2
  files_modified: 4
---

# Phase 19 Plan 01: Reference Docs, Profile Schema, and Agents — Infrastructure Contracts Summary

**One-liner:** Legal disclaimer updated with section 34d GewO insurance advisory notice, profile.json extended with typed insurance.policies[] schema, SKILL.md keyword map expanded to all 11 insurance types, and Field Extraction Schema added to health-insurance.md with field names matching profile.json exactly.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Update disclaimer, profile schema, SKILL.md keyword map | 548b5e3 | disclaimer.md, profile.json, SKILL.md |
| 2 | Add Field Extraction Schema to health-insurance.md | 56644b3 | health-insurance.md |

## What Was Built

### 1. disclaimer.md — section 34d GewO Notice (INFRA-02, INFRA-03)

Added insurance-specific advisory notice before the "You should:" list. The notice explicitly states Finyx is not a licensed Versicherungsmakler under section 34d GewO, that all guidance is criteria-based only, and directs users to a licensed advisor for binding product selection.

### 2. profile.json — insurance.policies[] Schema (INFRA-01)

Added two new top-level keys:
- `_insurance_schema` — documentation block (matches existing `_holdings_schema` pattern) documenting the type enum constraint, premium field separation rationale, coverage_amount nullability, doc_path convention, and coverage_components extraction pattern
- `insurance.policies[]` — typed array with all 14 required fields: id, type, provider, premium_monthly, premium_annual, coverage_amount, start_date, renewal_date, kuendigungsfrist_months, sonderkundigungsrecht, doc_path, coverage_components, notes, last_updated

The `type` field accepts one of 11 slugs: health, haftpflicht, hausrat, kfz, rechtsschutz, zahnzusatz, risikoleben, reise, fahrrad, kfz-schutzbrief, mietkaution.

### 3. SKILL.md — 11-Type Keyword Map Expansion

Phase 0 keyword map now covers all 11 insurance types with German aliases and English terms:
- 10 new keyword entries added (haftpflicht through mietkaution)
- AskUserQuestion singleSelect updated to list all 11 types with German name + English description
- Error handling updated to list all 11 available types
- Notes section clarified: other sub-skills will be added in Phases 21-22

### 4. health-insurance.md — Field Extraction Schema (ARCH-04 enablement)

New `## Field Extraction Schema` section added at the end of the file (before the source footnote). Section contains:
- Field extraction table with German policy label, type, and extraction notes for each field
- Health-specific extraction notes distinguishing GKV (Mitgliedsbescheinigung) from PKV (Versicherungsschein) document sources
- Field names match profile.json insurance.policies[] exactly: provider, premium_monthly, premium_annual, coverage_amount, start_date, renewal_date, kuendigungsfrist_months, coverage_components, exclusions

All existing sections 1-6 are preserved unchanged.

## Deviations from Plan

None — plan executed exactly as written.

## Key Links Verified

The key_links constraint from the plan is satisfied:
- `skills/profile/references/profile.json` field names (premium_monthly, premium_annual, coverage_amount, kuendigungsfrist_months) appear verbatim in the Field Extraction Schema table in `skills/insurance/references/germany/health-insurance.md`.

## Known Stubs

None. All four artifacts are complete and self-contained. The insurance.policies[] schema contains a placeholder entry (all zero/null values) as a schema template — this is intentional as it is a template file, not runtime data.
