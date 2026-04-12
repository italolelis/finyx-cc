# Phase 19: Reference Docs + Profile Schema + Agents - Research

**Researched:** 2026-04-12
**Domain:** Insurance reference documentation, profile schema extension, agent architecture
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
All implementation choices are at Claude's discretion — pure infrastructure phase. Use ROADMAP phase goal, success criteria, and codebase conventions to guide decisions.

Key constraints from requirements:
- ARCH-02: Generic research agent parameterized by insurance type
- ARCH-03: Portfolio agent for cross-type analysis (gaps, overlaps, total cost)
- ARCH-04: Document reader agent for PDF policy parsing
- ARCH-05: 11 per-type reference docs with coverage benchmarks, legal minimums, field extraction schemas
- INFRA-01: Profile schema extended with insurance.policies[] array
- INFRA-02: Legal disclaimer includes §34d GewO advisory-only notice
- INFRA-03: All output uses criteria-based recommendations, never specific product recommendations

### Claude's Discretion
All implementation choices — content of reference docs, agent prompt structure, field names in profile schema, output format of agents.

### Deferred Ideas (OUT OF SCOPE)
None — infrastructure phase.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| ARCH-02 | Generic research agent parameterized by insurance type | Existing finyx-insurance-research-agent.md is PKV-specific; must generalize to accept `insurance_type` param and load the matching reference doc |
| ARCH-03 | Portfolio agent for cross-type analysis (gaps, overlaps, total cost) | New agent; no existing analog in codebase. Pattern from calc-agent (read-only tools, stateless, XML-tagged output) applies |
| ARCH-04 | Document reader agent for PDF policy parsing | New agent; Read tool can ingest PDFs if path provided. Pattern: receives doc_path, extracts structured fields, returns XML output |
| ARCH-05 | 11 per-type reference docs with coverage benchmarks, legal minimums, field extraction schemas | health-insurance.md is the established template; 10 new docs needed (health migrated to sub-skill, original stays) |
| INFRA-01 | Profile schema extended with insurance.policies[] array | `.finyx/profile.json` has no insurance section; add `insurance.policies[]` with typed fields |
| INFRA-02 | Legal disclaimer includes §34d GewO advisory-only notice | `references/disclaimer.md` exists but lacks §34d GewO language |
| INFRA-03 | All output uses criteria-based recommendations, never specific product recommendations | Pattern already established in health sub-skill; research agent must carry forward for all types |
</phase_requirements>

---

## Summary

This phase creates all the static and structural infrastructure that Phases 20–22 will consume. It has no user-facing commands — the deliverables are reference documents (Markdown knowledge files), three new/updated agents, and a profile schema extension.

The codebase already has a mature reference doc pattern (`health-insurance.md`) and agent pattern (`finyx-insurance-research-agent.md`, `finyx-insurance-calc-agent.md`). Phase 19 is largely mechanical application of those patterns to 10 new insurance types, plus the creation of two genuinely new agents (portfolio analyzer and document reader) that have no existing analog.

The most consequential decisions are: (1) the schema design of `insurance.policies[]` since all downstream phases read it, and (2) how to make the research agent truly generic without losing the type-specific criteria that make it useful. The §34d GewO disclaimer update is a one-line change to `disclaimer.md` that must be verified complete before any sub-skill goes live.

**Primary recommendation:** Build reference docs first (they define the field extraction schemas that the document reader agent will use), then update the profile schema to match, then implement agents in dependency order (research generic → portfolio → document reader).

---

## Standard Stack

This project has no npm dependencies and no build step. All "libraries" are Markdown patterns.

### Core Patterns

| Artifact | Location Pattern | Purpose |
|----------|-----------------|---------|
| Reference doc | `skills/insurance/references/germany/{type}.md` | Static knowledge: benchmarks, legal minimums, field schemas |
| Agent | `skills/insurance/agents/finyx-insurance-{name}-agent.md` | Task-spawned specialist |
| Sub-skill | `skills/insurance/sub-skills/{type}.md` | Per-type advisor loaded by router |
| Profile JSON | `.finyx/profile.json` (user-owned) | Runtime user data |
| Disclaimer | `skills/insurance/references/disclaimer.md` | Shared legal footer |

### No Installation Required

All files are Markdown or JSON. No `npm install` step. Files are copied into `~/.claude/` at install time via `bin/install.js`.

---

## Architecture Patterns

### Reference Doc Structure (from health-insurance.md)

Every reference doc uses this YAML frontmatter:

```markdown
---
tax_year: 2025
country: germany
domain: {insurance-type}
last_updated: {date}
source: "{legal sources, §§ references}"
---
```

Sections per doc:
1. Overview — what the insurance covers, who needs it, legal minimum vs. recommended
2. Coverage Benchmarks — the criteria-based thresholds (e.g., Hausrat ≥ €650/m², Haftpflicht ≥ €5M)
3. Legal Minimums — statutory requirements where they exist (Kfz Haftpflicht is mandatory; most others are not)
4. Common Coverage Components — what a good policy includes vs. what is often excluded
5. Field Extraction Schema — the exact fields the document reader agent should extract from a PDF policy document
6. Keyword Map — German insurance terminology mapped to English for extraction (needed by document reader agent)
7. Cancellation Rules — Kündigungsfrist (typical 3 months), Sonderkündigungsrecht triggers
8. Sources — authoritative German sources (GDV, BaFin, Stiftung Warentest, Verbraucherzentrale)

### Agent Structure (from existing agents)

All agents follow:

```yaml
---
name: finyx-insurance-{name}-agent
description: {one sentence}
tools: Read, Grep, Glob[, WebSearch, WebFetch]
color: {color}
---
```

Body sections:
- `<role>` — identity, core job, stateless-by-design declaration, confidence flag rules, anti-hallucination rules
- `<execution_context>` — `${CLAUDE_SKILL_DIR}/references/` paths loaded at spawn time
- `<process>` — numbered phases
- `<output_format>` — XML-tagged output block the orchestrator parses

### Generic Research Agent Pattern (ARCH-02)

The existing `finyx-insurance-research-agent.md` is PKV-specific. The generic version must:

1. Accept `insurance_type` as a Task prompt parameter (e.g., `insurance_type: hausrat`)
2. Load the matching reference doc: `${CLAUDE_SKILL_DIR}/references/germany/{insurance_type}.md`
3. Read coverage benchmarks and legal minimums from that doc
4. Build type-specific WebSearch queries using the type's German terminology (from the keyword map in the reference doc)
5. Return criteria-based comparison output — what to look for, not which provider to buy from (§34d GewO)

The agent must NOT return specific tariff names or provider rankings — only criteria-based guidance (see INFRA-03). This distinguishes the generic insurance research agent from the PKV research agent (which was designed before this constraint was fully formalized and returns specific provider names — that agent is health-only and §34d GewO does not apply with the same force to advisory-only health comparisons, but all new types must use criteria-only output).

**Anti-pattern to avoid:** The existing PKV research agent returns 3 named providers. The new generic agent must NOT do this for non-health types. Return: coverage criteria checklist, benchmark thresholds to demand, red flags to watch for, and general market context (price ranges by segment). Never: "Buy from Allianz."

### Portfolio Agent Pattern (ARCH-03)

New agent with no existing analog. Read-only tools only (`Read, Grep, Glob`). Receives no Task prompt parameters beyond the profile path. Logic:

1. Read `insurance.policies[]` from `.finyx/profile.json`
2. Load all reference docs from `${CLAUDE_SKILL_DIR}/references/germany/` to get benchmarks
3. For each policy: check coverage against benchmark → adequate / below benchmark / unknown
4. For all types not in policies[]: flag as gap → mandatory (Kfz Haftpflicht) vs. essential vs. optional
5. Cross-policy overlap check: e.g., Hausrat + Haftpflicht often overlap on some coverage; Reise + Kfz-Schutzbrief may overlap
6. Total cost summary: sum all premiums, monthly and annual
7. Return XML-tagged output: `<portfolio_analysis>` with subsections for gaps, overlaps, adequacy, cost summary

### Document Reader Agent Pattern (ARCH-04)

New agent. Read-only tools (`Read, Grep, Glob`). Receives `doc_path` and `insurance_type` in Task prompt. Logic:

1. Load reference doc for `insurance_type` to get the Field Extraction Schema
2. Read the policy document at `doc_path` (Claude's Read tool handles PDF)
3. Extract each field from the schema: provider, policy_number, premium_monthly, premium_annual, coverage_amount, start_date, renewal_date, Kündigungsfrist, coverage_components[], exclusions[]
4. Map German terminology using the keyword map from the reference doc
5. Return structured output matching the `insurance.policies[]` schema so the orchestrator can write it to profile.json

### Profile Schema Extension (INFRA-01)

Current `.finyx/profile.json` has no `insurance` section. Add:

```json
"insurance": {
  "policies": [
    {
      "id": "string — generated UUID or slug",
      "type": "string — one of: health, haftpflicht, hausrat, kfz, rechtsschutz, zahnzusatz, risikoleben, reise, fahrrad, kfz-schutzbrief, mietkaution",
      "provider": "string",
      "premium_monthly": "number — EUR",
      "premium_annual": "number — EUR (may differ from monthly × 12 due to annual discount)",
      "coverage_amount": "number | null — EUR, null if coverage is service-based not sum-based",
      "start_date": "string — ISO 8601",
      "renewal_date": "string — ISO 8601 | null",
      "kuendigungsfrist_months": "number — default 3",
      "sonderkundigungsrecht": "boolean — whether Sonderkündigungsrecht is currently active",
      "doc_path": "string | null — relative path to policy PDF",
      "coverage_components": ["array of strings — what this policy covers"],
      "notes": "string | null — free text",
      "last_updated": "string — ISO 8601"
    }
  ]
}
```

Key design decisions:
- `type` is an enum constrained to the 11 known types (plus health which already exists)
- `premium_monthly` and `premium_annual` are separate fields because annual-pay discounts are common
- `coverage_amount` is nullable because some types (Rechtsschutz, Reise) provide unlimited or service-based coverage, not a fixed sum
- `doc_path` is a relative path from the user's project root (e.g., `.finyx/docs/hausrat-policy.pdf`)
- `coverage_components` is a string array because the document reader agent extracts these as free-text items

### §34d GewO Disclaimer Update (INFRA-02)

Current `disclaimer.md` has a general advisory disclaimer but no §34d GewO reference. The update must add:

```markdown
**Insurance advisory notice (§34d GewO):** Finyx is not a licensed insurance broker or Versicherungsmakler under §34d GewO. All insurance guidance is educational and criteria-based only. Finyx does not recommend specific insurance products, tariffs, or providers. For binding advice and product selection, consult a licensed Versicherungsberater or Versicherungsmakler.
```

This language must appear in `disclaimer.md` and be propagated to all insurance agents and sub-skills that load the disclaimer via `<execution_context>`.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead |
|---------|-------------|-------------|
| PDF parsing | Custom PDF parser | Claude's Read tool — it handles PDF natively |
| Type validation for insurance_type | Custom validator | Markdown prose constraint in agent `<role>` block |
| UUID generation for policy.id | Node.js UUID library | Bash `uuidgen` or slug from `{type}-{provider}` |
| Reference doc loading per type | Dynamic import system | Direct `Read ${CLAUDE_SKILL_DIR}/references/germany/${insurance_type}.md` in agent |

---

## Common Pitfalls

### Pitfall 1: Generic Agent Still Returns Provider Names

**What goes wrong:** Developer patterns from the PKV research agent (which names specific providers) bleed into the generic agent.
**Why it happens:** The PKV agent predates the §34d GewO constraint being fully formalized.
**How to avoid:** The generic research agent role block must explicitly state: "Never return specific provider or tariff names. Return criteria, benchmarks, and red flags only."
**Warning signs:** Output contains "Allianz", "HUK", "ERGO", "Debeka" etc.

### Pitfall 2: Reference Doc field schemas drift from profile.policies[] fields

**What goes wrong:** The document reader agent extracts fields that don't match `insurance.policies[]` field names, requiring manual mapping.
**Why it happens:** Reference docs are written first, profile schema is extended later, and field names diverge.
**How to avoid:** Write the Field Extraction Schema section in EVERY reference doc to match the `insurance.policies[]` JSON field names exactly. The profile schema is the source of truth.

### Pitfall 3: Kfz-Versicherung is Three Coverage Levels, Not One Policy

**What goes wrong:** A single `insurance.policies[]` entry for Kfz that mixes Haftpflicht, Teilkasko, and Vollkasko in one record.
**Why it happens:** In practice users have one Kfz contract, but the contract has multiple tiers.
**How to avoid:** One policy entry per Kfz contract, with `coverage_components` field capturing the tier (e.g., `["Haftpflicht", "Teilkasko"]`). The reference doc for Kfz must document all three tiers and their standard exclusions.

### Pitfall 4: Health type overlap with existing health-insurance.md

**What goes wrong:** Creating a new `health.md` reference doc for the generic type system while the old `health-insurance.md` still exists, causing the research agent to load the wrong doc.
**Why it happens:** The CONTEXT.md notes health "may need schema alignment" but doesn't explicitly direct renaming.
**How to avoid:** Keep `health-insurance.md` as-is (it's deeply linked by existing agents and sub-skills). The Field Extraction Schema section (new) should be added to `health-insurance.md` rather than creating a parallel file. The generic research agent will load `health-insurance.md` when `insurance_type == "health"`.

### Pitfall 5: portfolio agent tries to compare coverage across incompatible types

**What goes wrong:** Comparing coverage amounts across types where some have a monetary sum (Hausrat, Haftpflicht) and others are service-based (Rechtsschutz, Reise).
**Why it happens:** The portfolio agent sums or ranks policies naively.
**How to avoid:** Reference docs must specify `coverage_type: sum_based | service_based | unlimited`. Portfolio agent checks this flag before attempting monetary comparison.

### Pitfall 6: Disclaimer update not propagated

**What goes wrong:** `disclaimer.md` is updated with §34d GewO language but existing agents that load it via `<execution_context>` are not re-verified.
**Why it happens:** The execution_context loads the file at runtime, so propagation is automatic — BUT only if the path is correct and the agents actually load `disclaimer.md`.
**How to avoid:** Verify that EVERY agent file in `skills/insurance/agents/` includes `${CLAUDE_SKILL_DIR}/references/disclaimer.md` in its `<execution_context>`. The new agents created in this phase must also include it.

---

## Reference Doc Coverage Plan

### 10 New Reference Docs Required (health-insurance.md exists, add Field Extraction Schema to it)

| Type | German Name | Legal Minimum | Coverage Benchmark | Key Complexity |
|------|------------|--------------|-------------------|----------------|
| haftpflicht | Privathaftpflicht | None mandatory | ≥€5M Deckungssumme | Mietsachschäden exclusion common |
| hausrat | Hausratversicherung | None mandatory | ≥€650/m² Wohnfläche (Unterversicherungsverzicht) | Underinsurance is common failure |
| kfz | Kfz-Versicherung | Haftpflicht mandatory (§1 PflVG) | Fully comp (Vollkasko) for financed vehicles | SF-Klasse, Typklasse, Regionalklasse scoring |
| rechtsschutz | Rechtsschutzversicherung | None | Covers: Berufs-, Verkehrs-, Mieter-Rechtsschutz | Wartezeit (3 months) standard |
| zahnzusatz | Zahnzusatzversicherung | None | ≥80% Zahnersatz reimbursement; no Wartezeit ideally | GKV pays only 60% base; gap is large |
| risikoleben | Risikolebensversicherung | None | Sum = 3–5× annual income if dependents | Health underwriting like PKV |
| reise | Reiseversicherung | None | Kranken + Haftpflicht + Gepäck + Storno bundle | Coverage lapses after 42 days in some policies |
| fahrrad | Fahrradversicherung | None | Replacement cost coverage; e-bike specific riders | Often add-on to Hausrat |
| kfz-schutzbrief | Kfz-Schutzbriefversicherung | None | EU-wide towing, EU-wide breakdown assistance | Often included in ADAC or Kfz policy |
| mietkaution | Mietkautionsversicherung | None | Replaces cash deposit up to 3× net cold rent | Bank guarantee alternative; credit check required |

### Existing Doc to Extend

`health-insurance.md` — add `## Field Extraction Schema` section with the `insurance.policies[]` field names.

---

## Code Examples

### Reference Doc Frontmatter (pattern to replicate)

```markdown
---
tax_year: 2025
country: germany
domain: haftpflicht
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, §1 BGB (general liability), VVG (Versicherungsvertragsgesetz)"
---
```

### Field Extraction Schema Section (new section to add to all reference docs)

```markdown
## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Versicherer / Anbieter | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatsprämie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahresprämie | number | EUR |
| coverage_amount | Deckungssumme / Versicherungssumme | number | EUR |
| start_date | Versicherungsbeginn | ISO date | |
| renewal_date | Hauptfälligkeit / Verlängerungsdatum | ISO date | |
| kuendigungsfrist_months | Kündigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Leistungsumfang / Versicherungsschutz | string[] | Line items from coverage table |
| exclusions | Ausschlüsse / Nicht versichert | string[] | Exclusion list |
```

### Generic Research Agent Task Prompt Interface

```
insurance_type: hausrat
user_city: Munich
property_size_sqm: 75
current_premium_monthly: 18
current_provider: [optional]
current_year: 2026
```

### Portfolio Agent Output Format

```xml
<portfolio_analysis>

## Insurance Portfolio Overview

**Total monthly premium:** EUR XXX.XX
**Total annual premium:** EUR X,XXX.XX

### Coverage Adequacy

| Type | Status | Benchmark | Current Coverage | Action |
|------|--------|-----------|-----------------|--------|
| haftpflicht | OK | ≥€5M | €5M | — |
| hausrat | BELOW BENCHMARK | ≥€650/m² × 75m² = €48,750 | €40,000 | Increase Versicherungssumme |
| kfz | OK | Haftpflicht mandatory | Haftpflicht + Teilkasko | — |

### Gaps — Missing Insurance Types

| Type | Priority | Reason |
|------|----------|--------|
| zahnzusatz | HIGH — GKV covers only 60% base dental | No policy found in profile |
| rechtsschutz | MEDIUM — tenant rights coverage advised for renters | No policy found in profile |

### Overlaps — Potential Redundancy

| Overlap | Policies | Action |
|---------|----------|--------|
| Fahrrad covered by Hausrat add-on | hausrat policy includes Fahrrad-Zusatz, AND standalone fahrrad policy exists | Review if standalone is redundant |

### Cost Summary

| Type | Provider | Monthly | Annual |
|------|----------|---------|--------|
| haftpflicht | [provider] | EUR XX.XX | EUR XXX.XX |
| hausrat | [provider] | EUR XX.XX | EUR XXX.XX |
| **TOTAL** | | **EUR XXX.XX** | **EUR X,XXX.XX** |

</portfolio_analysis>
```

### Profile Schema Extension

```json
"insurance": {
  "policies": [
    {
      "id": "haftpflicht-huk-2024",
      "type": "haftpflicht",
      "provider": "HUK-COBURG",
      "premium_monthly": 8.50,
      "premium_annual": 102.00,
      "coverage_amount": 5000000,
      "start_date": "2024-01-01",
      "renewal_date": "2025-01-01",
      "kuendigungsfrist_months": 3,
      "sonderkundigungsrecht": false,
      "doc_path": ".finyx/docs/huk-haftpflicht-2024.pdf",
      "coverage_components": ["Personenschäden", "Sachschäden", "Mietsachschäden"],
      "notes": null,
      "last_updated": "2026-04-12"
    }
  ]
}
```

---

## Architecture Patterns

### Recommended Directory Structure After Phase 19

```
skills/insurance/
├── SKILL.md                          # router — update keyword map for all 11 types
├── agents/
│   ├── finyx-insurance-calc-agent.md       # existing — no change
│   ├── finyx-insurance-research-agent.md   # UPDATE — generalize to accept insurance_type
│   ├── finyx-insurance-portfolio-agent.md  # NEW (ARCH-03)
│   └── finyx-insurance-doc-reader-agent.md # NEW (ARCH-04)
├── references/
│   ├── disclaimer.md                 # UPDATE — add §34d GewO language
│   └── germany/
│       ├── health-insurance.md       # UPDATE — add Field Extraction Schema section
│       ├── haftpflicht.md            # NEW
│       ├── hausrat.md                # NEW
│       ├── kfz.md                    # NEW
│       ├── rechtsschutz.md           # NEW
│       ├── zahnzusatz.md             # NEW
│       ├── risikoleben.md            # NEW
│       ├── reise.md                  # NEW
│       ├── fahrrad.md                # NEW
│       ├── kfz-schutzbrief.md        # NEW
│       └── mietkaution.md            # NEW
└── sub-skills/
    └── health.md                     # existing — no change in Phase 19
```

### Anti-Patterns to Avoid

- **Type-specific logic in the generic agent:** The research agent must dispatch all type-specific knowledge to the reference doc. If you find yourself writing `if insurance_type == "hausrat"` logic in the agent, you've put that content in the wrong place — it belongs in `hausrat.md`.
- **Duplicating disclaimer content:** Don't copy §34d GewO text into agent role blocks. Update `disclaimer.md` once; agents load it via `<execution_context>`.
- **Writing files from agents:** Agents are stateless. The document reader agent returns structured output — the sub-skill orchestrator writes to `profile.json`.

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| PKV-specific research agent | Generic research agent parameterized by type | All 11 types get research capability without 11 agent files |
| No insurance section in profile | `insurance.policies[]` array | Portfolio agent and document reader can operate on real user data |
| General financial disclaimer only | §34d GewO specific notice | Legal compliance for German insurance advisory |
| Health reference doc only | 11 type-specific reference docs | Per-type benchmarks enable gap/adequacy detection |

---

## Open Questions

1. **SKILL.md keyword map expansion timing**
   - What we know: SKILL.md currently only maps `health`. Sub-skills for other types don't exist yet.
   - What's unclear: Should Phase 19 expand the keyword map for all 11 types, or leave that to Phases 21/22 when sub-skills are built?
   - Recommendation: Phase 19 should expand the keyword map entries for all 11 types and their German aliases. This is a forward declaration — the router will error gracefully if the sub-skill file doesn't exist yet, and adding keywords now prevents a partial-update across phases.

2. **`insurance_type` vs. `type` as the enum key in profile schema**
   - What we know: Convention in the rest of profile.json uses short keys.
   - What's unclear: Whether to use full German names (e.g., `"hausratversicherung"`) or short slugs (e.g., `"hausrat"`) for the `type` field.
   - Recommendation: Use short slugs (`hausrat`, `haftpflicht`, `kfz`, etc.) as they match the reference doc filenames and sub-skill filenames. Simpler path construction in agents.

3. **Document reader agent and non-PDF formats**
   - What we know: Claude's Read tool handles PDF. Some policies arrive as HTML or image scans.
   - What's unclear: How to handle image PDFs (scanned documents, no text layer).
   - Recommendation: Document in agent `<error_handling>` that scanned/image PDFs cannot be parsed and instruct user to provide a text-layer PDF or manually enter key fields via `/finyx:profile`.

---

## Environment Availability

Step 2.6: SKIPPED (no external dependencies — all artifacts are Markdown/JSON files created by Claude Code tools).

---

## Validation Architecture

`nyquist_validation` is set to `false` in `.planning/config.json`. Section skipped per instructions.

---

## Sources

### Primary (HIGH confidence)
- Existing codebase: `skills/insurance/references/germany/health-insurance.md` — reference doc structure and frontmatter schema
- Existing codebase: `skills/insurance/agents/finyx-insurance-research-agent.md` — agent structure, tool declarations, output format
- Existing codebase: `skills/insurance/agents/finyx-insurance-calc-agent.md` — stateless agent pattern, confidence flags
- Existing codebase: `skills/insurance/SKILL.md` — router pattern, sub-skill dispatch
- Existing codebase: `.finyx/profile.json` — current profile schema structure

### Secondary (MEDIUM confidence)
- REQUIREMENTS.md — canonical requirement definitions for ARCH-02 through ARCH-05, INFRA-01 through INFRA-03
- STATE.md accumulated context — §34d GewO constraint, Stiftung Warentest/Finanztip as primary sources, criteria-based language requirement

### Tertiary (LOW confidence — training knowledge)
- German insurance domain knowledge for coverage benchmarks (Hausrat ≥€650/m², Haftpflicht ≥€5M, etc.) — should be verified against GDV/Verbraucherzentrale sources when writing the reference docs
- §34d GewO text — should be verified against current GewO text when writing disclaimer update

---

## Metadata

**Confidence breakdown:**
- Reference doc structure: HIGH — direct pattern from existing health-insurance.md
- Agent architecture: HIGH — direct pattern from existing agents
- Profile schema design: HIGH — derived from requirements and existing JSON structure
- German insurance benchmarks in reference doc content: MEDIUM — training knowledge, verify against GDV/Verbraucherzentrale when authoring
- §34d GewO disclaimer language: MEDIUM — verify exact statutory language when writing

**Research date:** 2026-04-12
**Valid until:** 2026-05-12 (stable domain — no fast-moving libraries)
