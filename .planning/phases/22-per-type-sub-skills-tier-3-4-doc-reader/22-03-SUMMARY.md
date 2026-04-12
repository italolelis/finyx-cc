---
phase: 22-per-type-sub-skills-tier-3-4-doc-reader
plan: "03"
subsystem: insurance-skill
tags: [insurance, doc-reader, pdf-extraction, router, orchestrator]
dependency_graph:
  requires:
    - "22-01: kfz-schutzbrief and mietkaution sub-skills"
    - "19-01: reference docs and profile schema with documents.locations"
    - "19: finyx-insurance-doc-reader-agent"
  provides:
    - "doc-reader sub-skill orchestrating batch PDF extraction"
    - "router dispatch for doc/pdf/document keywords"
  affects:
    - "skills/insurance/SKILL.md — router keyword map and menu"
    - "skills/insurance/sub-skills/doc-reader.md — new orchestrator sub-skill"
tech_stack:
  added: []
  patterns:
    - "Orchestrator sub-skill: coordinates agent spawning, batch review, profile writes"
    - "AskUserQuestion confirmation gate before every profile write"
    - "Filename-based type auto-detection with longest-match priority"
    - "Full JSON read-update-write cycle to prevent profile corruption"
key_files:
  created:
    - skills/insurance/sub-skills/doc-reader.md
  modified:
    - skills/insurance/SKILL.md
decisions:
  - "doc-reader sub-skill is write-capable (unlike all type advisory sub-skills) — it writes insurance.policies[] and documents.locations.insurance"
  - "Filename-based type detection uses longest-match to disambiguate kfz vs kfz-schutzbrief"
  - "Full profile re-read before every write (not cached) to avoid stale-state overwrites"
  - "Phase 2 disclaimer placed after folder scan so users see file list before legal text"
metrics:
  duration_seconds: 111
  completed_date: "2026-04-12"
  tasks_completed: 2
  tasks_total: 2
  files_created: 1
  files_modified: 1
requirements_marked_complete: [OPT-02]
---

# Phase 22 Plan 03: Doc-Reader Sub-skill Summary

**One-liner:** Batch PDF policy extraction orchestrator with filename-based type detection, per-file agent spawning, batch confirmation table, and confirmed writes to insurance.policies[].

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Doc-reader sub-skill | 536852f | skills/insurance/sub-skills/doc-reader.md (created) |
| 2 | Router keyword update for doc-reader | 406dbed | skills/insurance/SKILL.md (modified) |

## What Was Built

### doc-reader.md (Task 1)

Six-phase orchestrator sub-skill:

- **Phase 0:** Resolves document folder from `documents.locations.insurance` in profile, or prompts user with AskUserQuestion and saves their answer (with confirmation) to the profile.
- **Phase 1:** Runs `ls *.pdf` in the configured folder; presents a multiSelect to choose which files to process.
- **Phase 2:** Emits FINYX banner + disclaimer.md + Document Reader Notice.
- **Phase 3:** Per-file type auto-detection (filename substring match, longest-match priority) with fallback to AskUserQuestion singleSelect. Spawns `finyx-insurance-doc-reader-agent` via Task tool per PDF. Collects `<doc_reader_result>` or records as failed extraction.
- **Phase 4:** Batch review table (file, type, provider, coverage, premium/mo, confidence). Failed Extractions section. Needs Review section for LOW CONFIDENCE results. AskUserQuestion multiSelect to confirm which to save.
- **Phase 5:** Per-confirmed-policy: re-read profile, check for duplicate (same type + provider), prompt on duplicate, append or replace, write full profile JSON.
- **Phase 6:** Summary of saved/skipped/failed with suggestion to run portfolio sub-skill.

### SKILL.md (Task 2)

Four additions to the router:
1. Keyword map: `"doc", "pdf", "document", "reader", "dokument", "police", "unterlagen" → doc-reader`
2. AskUserQuestion menu option: "Document reader (parse PDF policy documents into your profile)"
3. Phase 1 dispatch comment updated to include `doc-reader.md`
4. Error handling available types list updated with `doc-reader`

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None — doc-reader.md references the live `finyx-insurance-doc-reader-agent` agent and reads from actual `.finyx/profile.json`. No placeholder data or hardcoded stubs.
