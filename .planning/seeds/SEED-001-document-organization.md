---
id: SEED-001
status: dormant
planted: 2026-04-09
planted_during: v1.2 Health Insurance Advisor
trigger_when: next milestone (v1.3)
scope: Medium
---

# SEED-001: Financial document organization system

## Why This Matters

Users have scattered financial documents — insurance policies, tax returns, bank statements, investment account docs, health records. Finyx already reads `.finyx/profile.json` but has no structured place for supporting documents. When the user runs `/finyx:tax`, `/finyx:insurance`, or `/finyx:insights`, the agents could be much more accurate if they had access to actual documents (policy PDFs, account statements, tax returns) rather than relying solely on profile interview data.

The first command a user runs (`/finyx:profile` or a new `/finyx:init`) should scaffold this folder structure and explain how to use it. All subsequent commands and agents should reference these documents when available.

**Why:** This transforms Finyx from "interview-based advice" to "document-aware advice" — a fundamental accuracy and trust upgrade.

**How to apply:** When building this, the folder structure should be created by the profile/init command, with a README explaining what goes where. Agents should check for documents in these folders and use them as primary data sources over interview answers.

## When to Surface

**Trigger:** Next milestone (v1.3)

This seed should be presented during `/gsd:new-milestone` when the milestone scope matches any of these conditions:
- When profile/onboarding improvements are planned
- When document-based analysis features are discussed
- When accuracy improvements across advisors are a goal
- When a new `/finyx:init` command is being considered

## Scope Estimate

**Medium** — 1-2 phases. Phase 1: folder scaffolding + README generation in profile command. Phase 2: agent updates to read documents from known paths.

## Proposed Folder Structure

```
.finyx/
├── profile.json                    # Financial profile (existing)
├── insights-config.json            # Insights preferences (existing)
├── documents/
│   ├── README.md                   # How to use this folder
│   ├── insurance/
│   │   ├── health/                 # PKV/GKV policies, Bescheinigungen
│   │   ├── liability/              # Haftpflicht, Hausrat
│   │   └── life/                   # Lebensversicherung, BU
│   ├── accounts/
│   │   ├── statements/             # Monthly/yearly bank statements (PDF/CSV)
│   │   └── brokers/                # Broker account statements, Depotauszüge
│   ├── investments/
│   │   ├── portfolio/              # Current holdings, Wertpapierabrechnungen
│   │   └── pension/                # Riester/Rürup/bAV Bescheinigungen
│   ├── tax/
│   │   ├── 2025/                   # Tax return year
│   │   │   ├── Lohnsteuerbescheinigung.pdf
│   │   │   ├── Anlage-KAP.pdf
│   │   │   └── README.md           # What documents needed for this year
│   │   └── 2026/
│   │       └── README.md
│   ├── real-estate/
│   │   ├── contracts/              # Kaufverträge, Mietverträge
│   │   └── valuations/             # Gutachten, price lists
│   └── health/
│       └── policies/               # Health insurance policy docs
```

## Breadcrumbs

Related code and decisions in the current codebase:

- `commands/finyx/profile.md` — Profile creation command, natural place to add folder scaffolding
- `.finyx/profile.json` — Existing profile structure, would be extended with document paths
- `commands/finyx/tax.md` — Tax advisor could read actual Lohnsteuerbescheinigung
- `commands/finyx/insurance.md` — Insurance advisor could read actual policy documents
- `commands/finyx/invest.md` — Investment advisor could read actual Depotauszüge
- `agents/finyx-insurance-research-agent.md` — Research agent could cross-reference actual policy terms
- `.planning/PROJECT.md` — Out of Scope mentions "Budget tracking / expense categorization" but document organization is different (storage, not tracking)

## Notes

- User explicitly wants the tool to create this on first run and explain how to use it
- All commands and agents should check for and reference these documents
- Tax year subfolders should include a README explaining what documents are needed for that specific year's return
- The `/fin:budget` skill already reads bank statements — this folder structure gives it a canonical location
- Consider: should `/finyx:profile` detect existing documents and auto-populate profile fields from them?
