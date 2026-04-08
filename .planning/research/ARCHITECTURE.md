# Architecture Patterns — v1.2 Health Insurance Advisor

**Domain:** `/finyx:insurance` command integration into existing Finyx slash-command architecture
**Researched:** 2026-04-06
**Confidence:** HIGH (existing codebase reviewed directly; domain rules verified via web search)

---

## Integration Pattern

The health insurance command follows the same orchestrator-agent pattern as `/finyx:insights`: the command reads profile data, gates on completeness, spawns specialist agents via `Task`, synthesizes results, and outputs advisory text. No new architectural patterns are needed.

### Command Entry Point

`commands/finyx/insurance.md`

- `name: finyx:insurance`
- `allowed-tools: Read, Bash, Write, Task, AskUserQuestion`
- `execution_context`: loads `disclaimer.md`, `germany/health-insurance.md`, `profile.json`
- Outputs to terminal only (no file persistence by default); optionally saves summary to `.finyx/insurance-report.md` on user consent

---

## Component Boundaries

| Component | File | Responsibility | Communicates With |
|-----------|------|---------------|-------------------|
| Insurance command | `commands/finyx/insurance.md` | Orchestration, questionnaire, synthesis | Profile, agents, reference docs |
| PKV Provider Research agent | `agents/finyx-pkv-research-agent.md` | Live tariff and provider research | WebSearch, Read |
| Insurance advisor agent | `agents/finyx-insurance-advisor-agent.md` | GKV vs PKV cost comparison, tax calc, family impact, long-term projection | Read only (profile + ref docs) |
| Health insurance reference | `finyx/references/germany/health-insurance.md` | GKV/PKV rules, thresholds, Basisabsicherung deduction limits | Loaded by command and both agents |

Two agents, not one. The split is forced by tool scope: the PKV research agent needs `WebSearch` to fetch live tariffs; the insurance advisor agent must be stateless (no WebSearch) to produce deterministic, auditable calculations. This mirrors the separation between `finyx-allocation-agent` (deterministic math) and how research concerns are isolated in the real estate pipeline.

---

## Health Data Handling

**Decision: ephemeral by default, optional non-sensitive persistence with explicit consent.**

The health questionnaire collects pre-existing conditions, chronic diseases, and age/health flags for PKV risk surcharge estimation. This is the most sensitive data in the system.

| Option | Problem |
|--------|---------|
| Store in `profile.json` | Profile is read by all agents. Health conditions would leak into tax, pension, and investment contexts where they are irrelevant and create a privacy surface. |
| Separate `health-profile.json` | Adds a new read path to every agent that might need it; increases maintenance burden. |
| Ephemeral (in-session only) | No persistence risk. User re-enters if they re-run. Acceptable because the questionnaire is short (6-8 questions) and the command produces a durable output report. |

**Use ephemeral.** The command collects health data via `AskUserQuestion` in Phase 2, holds answers in working memory for the duration of the command, passes a sanitized summary to agents (see Structured Agent Input pattern below), and never writes raw health answers to disk.

**Optional non-sensitive persistence:** On completion, offer to save a `~/.finyx/insurance-config.json` with only non-sensitive questionnaire answers: `age`, `currently_insured_as` (GKV/PKV/none), `employment_type`, `family_members_to_cover`. Pre-existing conditions and chronic disease details stay ephemeral in all cases.

---

## Data Flow

```
/finyx:insurance
    |
    Phase 1: Validation
    |   Read .finyx/profile.json
    |   Extract: gross_income, tax_class, marginal_rate, family_status, children
    |   Gate: Germany active? (gross_income > 0 AND tax_class present)
    |   Gate: income above Versicherungspflichtgrenze? (PKV eligibility gate)
    |   Emit disclaimer before proceeding
    |
    Phase 2: Health Questionnaire (AskUserQuestion — ephemeral)
    |   Current insurance status (GKV / PKV / none)
    |   Employment type (employee / freelance / civil servant / student)
    |   Age (and partner age if family_status == married)
    |   Family members to cover (co-insurance needed?)
    |   Pre-existing conditions / chronic diseases (HIGH SENSITIVITY — never written to disk)
    |   Desired coverage level (basic GKV-equivalent / comprehensive / premium)
    |
    Phase 3: Parallel agent spawn (Task)
    |
    |-- finyx-insurance-advisor-agent
    |   Tools: Read only
    |   Input: profile fields + sanitized questionnaire summary (no raw health details)
    |   Reads: germany/health-insurance.md ref doc
    |   Output: GKV annual cost, PKV base cost estimate, employer share, Basisabsicherung
    |           tax deduction, Familienversicherung vs per-head PKV cost, 10yr + 30yr
    |           cost projection, eligibility flags (Anwartschaft if cross_border)
    |
    |-- finyx-pkv-research-agent
    |   Tools: WebSearch + Read
    |   Input: age, employment type, coverage level (no health details)
    |   Output: 3-5 PKV provider tariff ranges, Beitragsrückerstattung notes,
    |           Selbstbeteiligung tradeoffs, current Zusatzbeitrag comparisons
    |
    Phase 4: Synthesis
        Combine both agent outputs
        Produce GKV vs PKV comparison table
        Produce long-term cost projection table (10yr, 20yr, retirement age)
        Surface tax implication (Basisabsicherung deduction x marginal_rate = annual saving)
        Surface family impact (Familienversicherung free riders vs PKV per-head cost)
        Produce ranked recommendation with confidence level + disclaimer
        Offer to save non-sensitive config to .finyx/insurance-config.json
```

---

## Reference Doc: `finyx/references/germany/health-insurance.md`

Single file covering all rules the advisor agent needs. Sections:

| Section | Content |
|---------|---------|
| 1. Eligibility thresholds | Versicherungspflichtgrenze by year (2025: €73,800; 2026: €77,400); Beitragsbemessungsgrenze (2025: €66,150); freelance and civil servant special rules |
| 2. GKV cost formula | Base rate 14.6% (7.3% employee + 7.3% employer), Zusatzbeitrag range and current average, income cap, Familienversicherung rules |
| 3. PKV cost structure | Age/health-based pricing, Altersrückstellungen mechanics, historical avg increase rate (PKV ~3.1%/yr; GKV ~3.8%/yr), Beitragsrückerstattung, Selbstbeteiligung tradeoff |
| 4. Tax deductibility | Basisabsicherung portion fully deductible (Sonderausgabe); limits: €1,900 employees, €2,800 freelancers; Beitragsrückerstattung reduces deductible amount |
| 5. Family rules | Familienversicherung: free co-insurance for non-working spouse + children in GKV; PKV charges per person regardless of dependency status |
| 6. Expat rules | Anwartschaft (re-entry guarantee after time abroad), EU portability, re-entry conditions after GKV period |
| 7. Civil servant (Beihilfe) | State employer subsidizes 43-50% of premium; PKV strongly favored for Beamte regardless of income |
| 8. Tax year metadata | `tax_year: 2025` header for staleness detection — same pattern as `germany/tax-investment.md` |

No Brazil health insurance reference for v1.2. Brazil's SUS/plano de saúde landscape is structurally different (employer-provided plans dominate; individual PKV-equivalent rarely applies). Gate the command to Germany only with an explicit "Brazil health insurance advisory not yet available — coming in a future release" message.

---

## Profile Integration

The command reads from `profile.json` but writes nothing to it.

| Field | Used For |
|-------|----------|
| `countries.germany.gross_income` | GKV contribution calculation, Versicherungspflichtgrenze eligibility gate |
| `countries.germany.tax_class` | Gate: Germany active? |
| `countries.germany.marginal_rate` | Tax savings from Basisabsicherung deduction |
| `identity.family_status` | Familienversicherung vs PKV per-head cost |
| `identity.children` | Family coverage count |
| `identity.cross_border` | Trigger Anwartschaft notice for expats |
| `investor.income.total` | Cross-check against eligibility threshold |

Optional persistence: `.finyx/insurance-config.json` (non-sensitive fields only).

---

## Build Order

Each step unblocks the next. Reference doc is the hard dependency for both agents.

### Step 1 — Reference doc
`finyx/references/germany/health-insurance.md`

All rules, thresholds, formulas. Both agents load it via `@` include. Write this first; every downstream file references it. Getting thresholds wrong here propagates to all outputs.

### Step 2 — Calculation agent
`agents/finyx-insurance-advisor-agent.md`

Deterministic GKV vs PKV math, tax deduction, family impact, long-term projection. No WebSearch. Can be tested independently by feeding a profile snapshot and verifying the cost comparison output.

### Step 3 — Research agent
`agents/finyx-pkv-research-agent.md`

Live tariff research via WebSearch. Receives age, employment type, coverage level — no health conditions. Depends on Step 1 ref doc for framing context but not for calculations.

### Step 4 — Command
`commands/finyx/insurance.md`

Orchestrates Steps 2 + 3 agents. Owns the health questionnaire flow. Handles synthesis and disclaimer placement.

### Step 5 — Profile schema note (non-breaking)
Document the optional `insurance-config.json` shape in a comment in `immo/templates/config.json` or in the reference doc. No structural change to `profile.json` itself.

---

## Patterns to Follow

### Structured agent input (no raw health data)
The command passes a sanitized block to each agent, not the raw questionnaire answers. Pre-existing conditions become a boolean flag:

```
<insurance_context>
age: 38
employment: employee
family_members: spouse_38 child_6
current_insurance: GKV
desired_coverage: comprehensive
pkv_eligible: true
preexisting_surcharge_applicable: true
</insurance_context>
```

The advisor agent applies a surcharge estimate from the reference doc (typically +20-50% on base tariff for pre-existing conditions) without knowing the specific condition. The command holds the raw condition details in working memory only for surfacing relevant caveats in the synthesis phase.

### Disclaimer before questionnaire
Emit the legal disclaimer at the end of Phase 1 (before Phase 2 health questionnaire). The user must see the disclaimer before providing sensitive health data. This is the existing pattern in all advisor commands.

### Staleness detection
Check `tax_year` in the health-insurance.md header. Warn if stale using the same pattern as `pension.md` Phase 1.

### No Brazil gate logic beyond notice
The command checks `identity.cross_border` and emits a one-line notice if true: "Note: Brazil health insurance advisory is not yet available." Then proceeds with Germany analysis. No abort, no complex branching.

---

## Anti-Patterns to Avoid

### Do not store pre-existing conditions in any file
Any file write containing health conditions creates GDPR and liability exposure. Raw health answers are ephemeral only, always.

### Do not ask health questions before showing the disclaimer
Legal disclaimer must appear before the user provides sensitive health data. Phase 1 ends with the disclaimer; Phase 2 begins the questionnaire.

### Do not hardcode thresholds or rates
Versicherungspflichtgrenze, GKV contribution rate, Zusatzbeitrag, Beitragsbemessungsgrenze, and Basisabsicherung deduction limits change annually. All rates live in `health-insurance.md`, never in the command or agents.

### Do not attempt Brazil health insurance in v1.2
Defer to a future milestone. A one-line notice in the command output is the correct handling.

### Do not create a third agent for tax math
The Basisabsicherung deduction calculation (premium * deductible_fraction * marginal_rate) is simple enough for the insurance advisor agent to handle inline. A dedicated tax agent for this would over-fragment the architecture.

### Do not give the PKV research agent raw health details
The research agent needs age and employment type to find realistic tariff ranges. It does not need condition details, and providing them would create a WebSearch query containing sensitive health information.

---

## Scalability Considerations

| Concern | v1.2 | Future |
|---------|-------|--------|
| Brazil health insurance | One-line "not yet available" notice | Add `finyx/references/brazil/health-insurance.md` + conditional country block |
| Civil servant (Beihilfe) path | Covered in ref doc section 7; advisor agent uses conditional branch | No structural change needed |
| Supplemental insurance (dental, liability, disability) | Out of scope per PROJECT.md | Separate command; `finyx:insurance` stays PKV/GKV focused |
| PKV provider data freshness | WebSearch in research agent | No architectural change needed |
| Re-running without re-answering questionnaire | Non-sensitive fields in `insurance-config.json` | On re-run: load config, skip those questions, re-ask health questions only |

---

## Sources

- Codebase: `agents/finyx-tax-scoring-agent.md`, `commands/finyx/insights.md`, `commands/finyx/pension.md` — HIGH confidence (direct read)
- GKV/PKV thresholds 2025/2026: [Feather Insurance](https://feather-insurance.com/blog/private-health-insurance-threshold-yearly-changes), [myhealthcarebroker.com 2026](https://www.myhealthcarebroker.com/blog/jahresarbeitsentgeltgrenze-2026-private-health-insurance), [Ogletree 2026 increases](https://ogletree.com/insights-resources/blog-posts/germany-increases-social-security-contribution-and-compulsory-insurance-ceilings-effective-january-1-2026/) — MEDIUM confidence
- GKV contribution mechanics: [steuergo.de 2025](https://www.steuergo.de/en/fag/2025/399/how_is_my_contribution_to_statutory_health_insurance_calculated_in_2025), [acciyo.com payroll 2025](https://www.acciyo.com/germany-payroll-taxes-employee-and-employer-contributions-2025-guide/) — MEDIUM confidence
- PKV cost trends: [germanpedia.com PKV cost development](https://germanpedia.com/private-health-insurance-cost-development/), [Feather — is PKV worth it 2026](https://feather-insurance.com/blog/private-health-worth-it) — MEDIUM confidence
- Tax deductibility: [germanpedia.com tax-deductible](https://germanpedia.com/tax-deductible-health-insurance-costs-germany/), [germantaxes.de deductions](https://germantaxes.de/deducting-insurance-from-tax/) — MEDIUM confidence
