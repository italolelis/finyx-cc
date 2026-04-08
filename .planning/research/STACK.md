# Technology Stack

**Project:** Finyx — Personal Finance Advisor CLI
**Milestone:** v1.2 Health Insurance Advisor — `/finyx:insurance` command
**Researched:** 2026-04-06
**Confidence:** HIGH

---

## Architecture Constraint (unchanged from v1.0/v1.1)

All logic lives in Markdown prompts. Single Node.js installer handles distribution. Zero runtime dependencies — absolute constraint. No new npm packages. No APIs requiring API keys for this feature.

The "stack" question for v1.2 is: which values belong in **versioned reference docs** (stable, annually updated), which are **computed inline by Claude** (formulas and arithmetic), and which need **live web search** (provider tariffs that change frequently)?

---

## What v1.2 Adds vs v1.1

| Capability | Need | Solution |
|------------|------|---------|
| GKV cost calculation | Income-based statutory rate + Zusatzbeitrag per fund + Pflegeversicherung | Reference doc with 2026 base rates + formula; live search for fund-specific Zusatzbeitrag |
| PKV cost estimation | Age/health-based tariff ranges by profile segment + employer share | Reference doc with age-bracket ranges; live search for provider tariffs |
| Health risk questionnaire | Pre-existing condition surcharges (Risikozuschlag), exclusion likelihood | Reference doc with surcharge categories and typical loading factors |
| Family impact analysis | Familienversicherung vs separate PKV costs per family structure | Reference doc with 2026 family cost formulas |
| Tax deduction calculation | PKV Basisabsicherung Sonderausgaben limits (€1,900 employees / €2,800 self-employed) | Reference doc, computed inline |
| Long-term projection | PKV age-curve growth vs GKV income-tracking growth over 10/20/30 years | Formula constants in reference doc, computed inline by Claude |
| Provider research agent | Live tariff ranges, Beitragsrückerstattung offers, Selbstbeteiligung options | Live web search (WebSearch + WebFetch tools) |
| Expat considerations | Anwartschaft, JAEG threshold, EU portability | Reference doc |

---

## v1.2 Stack Additions

### 1. Health Insurance Reference Doc (new: `finyx/references/germany/health-insurance.md`)

**The primary artifact for this milestone.** All stable 2026 values encoded here. This file gets annual review like `tax-rules.md`.

#### Values to encode (all verified, HIGH confidence):

**GKV contribution framework:**

| Value | 2026 | Notes |
|-------|------|-------|
| Base KV rate (allgemeiner Beitragssatz) | 14.6% | Split 7.3% employee / 7.3% employer |
| Average Zusatzbeitrag | 2.9% | Half employee / half employer; fund-specific (2.18%–4.4% range) |
| Beitragsbemessungsgrenze (BBG) monthly | €5,812.50 | Annual: €69,750 — contributions capped at this income |
| Pflegeversicherung (with children) | 3.6% | 1.7% employer + 1.9% employee |
| Pflegeversicherung (childless, 23+) | 4.2% | Extra 0.6% surcharge paid entirely by employee |
| PV reduction per child (2nd+, under 25) | −0.25% per child | Minimum 2.4% at 5+ children |
| JAEG (switch threshold, employed) | €77,400 gross/year | For new switchers; existing PKV holders: €69,750 threshold |
| Maximum GKV monthly contribution | ~€1,105 (KV) + ~€244 (PV) | Combined at BBG ceiling |

**Employer contribution (Arbeitgeberzuschuss) limits for PKV employees:**

| Component | 2026 monthly cap |
|-----------|-----------------|
| Private Krankenversicherung | €508.59 |
| Private Pflegeversicherung | €104.63 (€75.56 Saxony) |
| Combined total | €613.22 |

Employer pays half of what they would contribute for GKV, capped at the above limits. This is a core input for the break-even analysis.

**PKV premium benchmarks by age bracket (approximate ranges, verified from multiple broker sources):**

| Age at entry | Monthly PKV (comprehensive, single) | Notes |
|-------------|--------------------------------------|-------|
| 25 | €250–350 | Healthy, no pre-existing conditions |
| 30 | €350–500 | Standard comprehensive tariff |
| 35 | €450–650 | Higher base, depends on health |
| 40 | €550–800 | Significant age loading begins |
| 45 | €700–1,000 | Aging provisions less effective |
| 50 | €900–1,200+ | Premium increases accelerate |

These are starting-point brackets for the command to use when provider live search is unavailable. Command should always pursue live search for actual tariffs.

**Premium increase risk (for long-term projection):**
- Historical average PKV annual increase: 4–8% (varies by insurer and tariff)
- 2026 actual increases: 7–12% (elevated cycle due to healthcare cost surge)
- Conservative projection assumption: 5% annual growth (encodes in projection formula)
- GKV comparison growth: indexed to BBG increase (typically 3–5%/year with income growth)

**Family cost formulas (PKV vs GKV):**

GKV family (Familienversicherung):
- Non-working spouse earning < €603/month: FREE (covered under contributor)
- Children: FREE (all covered under one GKV premium)
- Total: single-contributor premium only

PKV family:
- Non-working partner: separate PKV policy (€300–600/month) OR GKV freiwillige Versicherung (~€265–285/month child)
- Children: €150–200/month each in PKV
- Break-even: family structure heavily favors GKV for 2+ children and non-working partner

**Tax deduction (Basisabsicherung):**
- Employees: Vorsorgeaufwendungen cap €1,900/year. PKV Basisabsicherung contributions fully deductible up to this cap.
- Self-employed: cap €2,800/year.
- The Basisabsicherung portion is typically 70–90% of the total PKV premium (excess covers extras like single room or chief physician).
- Command instruction: apply marginal tax rate from `profile.json` to calculate net PKV cost after deduction.

**Health risk loading categories:**

| Condition category | Typical Risikozuschlag | Exclusion risk |
|-------------------|------------------------|----------------|
| Well-controlled chronic (e.g. diabetes Type 2) | 20–30% | Low |
| Cardiovascular history | 40–50% | Medium |
| Mental health treatment (ongoing/recent 5yr) | 30–50% or rejection | High |
| Musculoskeletal (back, joints) | 10–20% | Low |
| Cancer history (in remission 5+ years) | 30–60% | Medium |
| No significant history | 0% | None |

**Anwartschaft (expat dormant coverage):**
- Freezes PKV policy while abroad for a small monthly fee (typically €30–80/month)
- Preserves age entry premium on reactivation — no new health check required
- Recommended for any PKV holder considering working abroad for < 5 years

---

### 2. PKV Provider Research Agent (new: `agents/finyx-insurance-agent.md`)

**Purpose:** Live search for actual tariffs from major PKV providers. Called by `/finyx:insurance` when the user wants provider-specific quotes.

**Tools:** `WebSearch`, `WebFetch` (already available in agent architecture).

**Search strategy (encode in agent prompt):**

Primary scrape targets (no API — web search + WebFetch):
- `check24.de/private-krankenversicherung/` — aggregator tariff data
- `ottonova.de/en/tariffs` — digital-first PKV, transparent pricing
- `debeka.de` — stable premium history, popular for employees
- `barmenia.de` — competitive on young/healthy profiles
- `huk24.de/pkv` — cost-competitive tariffs
- `feather-insurance.com` — expat-focused with English UI

**No API exists for PKV tariff data.** BaFin publishes mortality tables and regulatory data, but not tariff pricing. All provider tariff data is behind quote calculators that require age/health input. The agent must:
1. Search for `[provider] PKV Beiträge [age] 2026` to find published benchmark ranges
2. Use company-published example tariffs for specific age brackets
3. Always caveat that actual quotes require provider health questionnaire

**GKV fund-specific Zusatzbeitrag** is publicly available and scrapeable. Sources:
- `krankenkasseninfo.de/krankenkassen/zusatzbeitrag/` — live table, all funds, current rates
- `gkv-zusatzbeitrag.de` — comparison tool
- `covago.de/uebersicht-gkv-zusatzbeitraege-2026/` — verified table with sources cited

Agent can fetch current Zusatzbeitrag for any named fund via WebFetch on these URLs.

---

### 3. Profile Integration (additions to existing `profile.json` schema)

New fields needed under a new `insurance` section:

```json
{
  "insurance": {
    "current_type": "gkv|pkv|none",
    "current_provider": "TK|AOK|Barmer|Debeka|...",
    "current_monthly_cost": null,
    "employment_status": "employed|self_employed|civil_servant|student|retired",
    "health_flags": {
      "chronic_conditions": false,
      "mental_health_treatment_5yr": false,
      "cardiovascular_history": false,
      "musculoskeletal_issues": false,
      "other_significant": false
    },
    "family": {
      "partner_insured_separately": false,
      "partner_income": null,
      "children_count": 0,
      "children_under_25": 0
    },
    "expat_status": {
      "currently_abroad": false,
      "has_anwartschaft": false,
      "years_in_germany": null
    }
  }
}
```

**Source for fields:** Derived from the command's health questionnaire, written to profile.json for cross-advisor use. The `/finyx:insights` command can then flag insurance optimization opportunities.

---

### 4. Long-Term Projection Formula (inline, no library)

Two competing trajectories:

**PKV trajectory:**
```
PKV_t = PKV_0 × (1 + r_pkv)^t
```
Where:
- `PKV_0` = current/estimated monthly PKV premium
- `r_pkv` = 0.05 (5% annual growth, conservative; encode in reference doc)
- `t` = years
- Subtract employer contribution cap (€508.59/month, grows with BBG at ~3%/year)
- Add net-of-tax benefit: reduce by `marginalRate × 0.85 × PKV_0 / 12` (Basisabsicherung portion)

**GKV trajectory:**
```
GKV_t = min(income_t × rate_total, BBG_t × rate_total)
```
Where:
- `income_t = income_0 × (1 + r_income)^t` (use `goals.income_growth_rate` from profile or default 2%)
- `rate_total = 0.073 + (Zusatzbeitrag/2) + PV_employee_rate`
- `BBG_t = BBG_0 × (1 + r_bbg)^t` (BBG grows ~3%/year historically)
- Employee pays half: `GKV_cost_t = GKV_t × 0.5`

Claude computes both trajectories over 10/20/30 year horizons and outputs a comparison table with crossover year.

---

## Reference Doc Architecture (updated)

```
finyx/references/
  germany/
    tax-investment.md
    tax-rules.md
    pension.md
    brokers.md
    health-insurance.md          # NEW for v1.2 — core artifact
  brazil/
    tax-investment.md
    pension.md
    brokers.md
  insights/
    scoring-methodology.md
    projection-formulas.md
```

Brazil has no private/public split analogous to PKV/GKV. The `/finyx:insurance` command is Germany-only in v1.2. Country gate in command prompt: "This command currently supports Germany only. Brazil health insurance advisory coming in a future release."

---

## What Needs Live Search vs Reference Doc

| Data | Source | Rationale |
|------|--------|-----------|
| Base GKV rate (14.6%) | Reference doc | Set by law, changes only via legislation (rare) |
| Average Zusatzbeitrag (2.9%) | Reference doc | Annual government estimate, updated once/year |
| Specific fund Zusatzbeitrag | Live search | Funds set their own rates, change mid-year |
| BBG, JAEG thresholds | Reference doc | Set annually via Sozialversicherungsrechengrößen-Verordnung |
| PKV age-bracket ranges | Reference doc (broad) + live search | Reference gives ballpark; agent searches actual tariffs |
| Provider Beitragsrückerstattung offers | Live search | Changes per provider/year |
| PKV premium increase history | Reference doc | Historical average; specific insurer increases via live search |
| Tax deduction caps (€1,900/€2,800) | Reference doc | Set by EStG, changes rarely |
| Pflegeversicherung rates | Reference doc | Set by law |
| Employer subsidy caps | Reference doc | Derived from BBG formula; updated annually |

---

## What NOT to Add

| Option | Why not |
|--------|---------|
| Insurance comparison API (Check24, Verivox) | No public API; scrapers violate ToS; reference doc + WebSearch achieves 90% of value |
| Actuarial pricing library | PKV actuarial math (Sterbetafeln) is not needed — age-bracket ranges suffice for advisory |
| BaFin mortality table integration | BaFin publishes tables for insurers setting reserves, not for consumer tools |
| Any npm package | Zero runtime dependency constraint is absolute |
| Brazil PKV equivalent | No meaningful analog exists; scope to Germany only in v1.2 |

---

## Unchanged from v1.1

- All four market data APIs (Finnhub, brapi.dev, Bundesbank SDMX, OpenFIGI)
- Node.js installer, CommonJS, zero runtime deps
- Claude Code slash-command architecture
- `.finyx/profile.json` as single source of truth for user data
- `@path` include pattern for reference doc injection

---

## Sources

- 2026 GKV Beitragsbemessungsgrenze and JAEG thresholds: https://www.privat-patienten.de/verbraucher/versicherungspflichtgrenze/ (HIGH)
- 2026 employer PKV subsidy caps: https://www.privat-patienten.de/verbraucher/arbeitgeberzuschuss-private-krankenversicherung/ (HIGH)
- GKV Zusatzbeitrag range 2026 (2.18%–4.4%): https://www.krankenkasseninfo.de/krankenkassen/zusatzbeitrag/ (HIGH)
- Pflegeversicherung 2026 rates (3.6%/4.2%): https://www.gkv-spitzenverband.de/pflegeversicherung/pv_grundprinzipien/pflege_beitragssatz/beitragssatz.jsp (HIGH)
- PKV premium increase projections 2026: https://www.ottonova.de/en/premium-increase2026 (MEDIUM)
- Pre-existing condition surcharge ranges: https://feather-insurance.com/blog/pre-existing-conditions (MEDIUM)
- Family cost comparison PKV vs GKV: https://schlemann.com/krankenversicherung/private-krankenversicherung-mit-familie/ (MEDIUM)
- Anwartschaft mechanics: https://www.privat-patienten.de/lexikon/begriff/auslandsaufenthalt-dauerhaft/ (HIGH)
- PKV tax deductibility caps: https://www.versicherungenmitkopf.de/private-krankenversicherung/steuerlich-absetzbar (HIGH)
- BaFin PKV mortality table 2026: https://www.bafin.de/SharedDocs/Veroeffentlichungen/DE/Meldung/2025/meldung_2025_06_23_PKV_Sterbetafeln.html (HIGH — regulatory reference)
- Digital data exchange PKV→BZSt 2026: https://www.grantthornton.de/en/insights/2026/electronic-data-exchange-for-private-health-insurance-from-2026/ (HIGH)
- Live GKV fund Zusatzbeitrag comparison: https://www.gkv-zusatzbeitrag.de/en/ (HIGH)

---
*Stack research for: Finyx v1.2 Health Insurance Advisor*
*Researched: 2026-04-06*
