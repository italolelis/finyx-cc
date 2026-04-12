---
task_id: 260412-lf2
title: Make broker handling agnostic — remove hardcoded broker bias
type: quick
completed: 2026-04-12
duration: ~15m
tasks_completed: 3
tasks_total: 3
files_modified: 2
---

# Quick Task 260412-lf2: Make Broker Handling Agnostic

broker.md now discovers brokers dynamically via WebSearch using reference docs as a baseline, scores against user answers with criteria-based weighting, and saves preferred brokers back to profile.json.

## Tasks

| Task | Name | Status | Commit |
|------|------|--------|--------|
| 1 | Rewrite broker.md — agnostic discovery with WebSearch | Done | cfb7d46 |
| 2 | Update help.md — remove specific broker names | Done | bca5ab3 |
| 3 | Verify invest.md is already agnostic (read-only) | Done | no changes needed |

## Changes Made

### commands/finyx/broker.md (rewrite)

- Added Phase 2.5: reads existing brokers from profile and asks comparison scope
- Phase 3 and 4: load reference baseline then WebSearch to discover additional brokers
- Phase 5: replaced hardcoded decision matrices with dynamic criteria-based scoring
- Added Phase 5.5: saves preferred broker(s) to profile.json (append-only)
- Phase 6: generic table headers, no specific broker names
- Notes updated to explain live discovery and criteria-based scoring

### commands/finyx/help.md (targeted edits)

- Command table row updated to "with live market discovery"
- Covers bullets: specific broker names replaced with "live discovery + baseline reference data"
- Added bullet for web search broker detection

### commands/finyx/invest.md (read-only verification)

No changes needed — already agnostic.

## Deviations from Plan

None — plan executed exactly as written.

## Verification

- No hardcoded broker names in broker.md or help.md: PASS
- WebSearch count in broker.md: 9 (>= 2 required)
- baseline count in broker.md: 16 (>= 1 required)
- Reference docs unchanged: PASS

## Self-Check: PASSED
