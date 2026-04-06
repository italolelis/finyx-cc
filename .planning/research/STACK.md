# Technology Stack

**Project:** Finyx — Personal Finance Advisor CLI
**Milestone:** v1.1 Financial Insights Dashboard — `/fin:insights` command
**Researched:** 2026-04-06
**Confidence:** HIGH

---

## Architecture Constraint (unchanged from v1.0)

This is NOT a traditional application stack. All logic lives in Markdown prompts. A single Node.js installer handles distribution. Agents spawn via Claude Code's Task tool. **The zero-runtime-dependency constraint is absolute** — no `npm install` additions.

The "stack" question for v1.1 is: what **patterns and reference structures** enable a cross-advisor aggregation and scoring layer without touching the runtime?

---

## What v1.1 Adds vs v1.0

v1.0 established: data APIs, country tax reference docs, agent command pattern.

v1.1 needs four new capabilities:

| Capability | Need | Solution |
|------------|------|---------|
| Cross-advisor data aggregation | Read outputs from tax, invest, pension commands | Read `.finyx/profile.json` + cached advisor outputs via `@` includes |
| Financial health scoring | Score income allocation, savings rate, tax efficiency | Scoring rubric encoded in reference doc + inline Claude arithmetic |
| Net worth projection | FV calculations with compound growth | Formula constants in reference doc, calculation by Claude in-prompt |
| CLI visualization | Progress bars, allocation charts in terminal | Unicode block characters in Markdown output — no library needed |

---

## v1.1 Stack Additions

### 1. Advisor Output Cache (new file: `.finyx/insights-cache.json`)

**Purpose:** Avoid re-running 4 specialist commands each time `/fin:insights` is called. Each advisor command writes a structured summary block to a shared cache file on consent.

**Pattern:** Same pattern as `.finyx/profile.json` — plain JSON, written by commands, read by insights command via `@.finyx/insights-cache.json`.

**Schema (new, to be implemented):**

```json
{
  "version": "1.0.0",
  "updated": "2026-04-06T12:00:00Z",
  "tax": {
    "cached_at": "ISO date",
    "country": "germany",
    "sparerpauschbetrag_used": 0,
    "sparerpauschbetrag_remaining": 2000,
    "tax_efficiency_score": null,
    "optimization_gaps": []
  },
  "invest": {
    "cached_at": "ISO date",
    "total_portfolio_eur": null,
    "allocation_by_class": {},
    "drift_from_target": {},
    "rebalancing_needed": false
  },
  "pension": {
    "cached_at": "ISO date",
    "projected_monthly_income_eur": null,
    "pension_gap_eur": null,
    "riester_zulagen_unclaimed": false
  }
}
```

**Why not re-read advisor outputs inline:** Advisor commands produce conversational text, not machine-readable JSON. The cache schema forces structured output. This is the correct separation: advisors give advice, the cache extracts the key numbers.

**Confidence:** HIGH — follows established `.finyx/profile.json` pattern exactly.

### 2. Financial Health Scoring Reference Doc (new: `finyx/references/insights/scoring-methodology.md`)

**Purpose:** The `/fin:insights` command needs a rubric for scoring. Rather than hardcoding benchmarks into the command prompt (fragile, hard to maintain), encode them in a versioned reference doc.

**Content to encode:**

| Dimension | Benchmark | Source | Confidence |
|-----------|-----------|--------|------------|
| Savings rate | 20% of net income (50/30/20 rule) | CFPB, NerdWallet, Bloomberg WealthScore | HIGH |
| Emergency fund | 3–6 months of expenses | Standard personal finance guidance | HIGH |
| Debt-to-income | Below 36% considered healthy | Bloomberg WealthScore, FHN methodology | HIGH |
| Investment rate | 10–15% of gross income as a floor | Standard wealth-building guidance | MEDIUM |
| Tax efficiency (DE) | Sparerpauschbetrag utilization ≥ 80% | Derived from German tax rules | HIGH |
| Pension gap | Target 70–80% of final salary in retirement | Standard replacement rate guidance | MEDIUM |

**Scoring scale:** 1–10, following Bloomberg WealthScore's unweighted average of benchmark sub-scores. Each dimension scores independently; insights command averages them with narrative context.

**Why a reference doc, not hardcoded:** Tax thresholds change annually. The 50/30/20 benchmarks differ for high-income earners in high-cost cities (relevant for a €210k income in Germany where housing costs alone exceed 30%). The reference doc gets the same annual review treatment as tax-rules.md.

### 3. Net Worth Projection Formula (encoded in reference doc)

**No library needed.** The two required formulas are:

```
FV of current net worth:
  NW_future = NW_now × (1 + r)^t

FV of recurring monthly contributions:
  FV_annuity = C × [((1 + r)^t - 1) / r]

Total projected net worth = NW_future + FV_annuity
```

Where `r` = monthly return rate, `t` = months to horizon, `C` = monthly savings + investment contributions.

Claude performs this arithmetic inline given profile data. No `financejs` or external library is needed. The formula is deterministic and simple — adding a dependency for two equations would violate the zero-dependency constraint for no benefit.

**Confidence:** HIGH — standard time-value-of-money formulas, verified against Finance.js source and compound interest references.

**Default return rate assumptions (to encode in reference doc):**

| Asset mix | Annual return assumption | Rationale |
|-----------|-------------------------|-----------|
| Conservative (bonds/cash heavy) | 3.5% | Aligned with current EUR risk-free + small equity premium |
| Moderate (balanced) | 5.5% | MSCI World historical real return minus inflation |
| Aggressive (equity-heavy) | 7.5% | MSCI World 30-year nominal average |

These map to `goals.risk_tolerance` from `profile.json`.

### 4. CLI Visualization — Unicode Block Characters (no library)

**Purpose:** Display income allocation breakdown and net worth projections visually in terminal output.

**Approach:** Claude outputs Unicode block characters directly in Markdown. No library, no rendering engine.

**Patterns to encode in command prompt:**

Horizontal bar (allocation):
```
Savings    ████████████░░░░░░░░  18% (target: 20%)
Needs      ██████████████████░░  45% (target: 50%)
Wants      ████████░░░░░░░░░░░░  22% (target: 30%)
```

Sparkline (goal progress over time):
```
Net worth trajectory (5yr): ▁▂▃▄▅▅▆▇█
```

Unicode blocks used: `▁▂▃▄▅▆▇█` (U+2581–U+2588) for sparklines, `█` + `░` for bar fills.

**Why no library:** The existing architecture has zero runtime dependencies. The `sparklines` Python library and similar tools require a runtime. Claude can generate these characters natively from numeric data — it is an LLM, not a dumb formatter. This is the correct approach for a prompt-based system.

**Confidence:** HIGH — Unicode block character rendering is supported in all modern terminals. Verified in existing Finyx command output patterns.

---

## Integration Points with Existing Commands

| Existing command | What insights reads from it | Integration mechanism |
|-----------------|----------------------------|-----------------------|
| `/finyx:profile` | `profile.json`: income, goals, risk tolerance, liquid assets | `@.finyx/profile.json` include (already established) |
| `/finyx:tax` | Sparerpauschbetrag remaining, optimization gaps | New `insights-cache.json` (written by tax command on consent) |
| `/finyx:invest` | Portfolio total, allocation drift, rebalancing status | New `insights-cache.json` (written by invest command on consent) |
| `/finyx:pension` | Pension gap, unclaimed Zulagen | New `insights-cache.json` (written by pension command on consent) |

The insights command reads all four sources. If cache entries are absent (user hasn't run an advisor command yet), insights degrades gracefully: it performs partial analysis on profile data alone and flags which advisors to run for fuller picture.

---

## Existing Stack (unchanged from v1.0)

### Market Data APIs

| API | Purpose | Auth | Confidence |
|-----|---------|------|------------|
| Finnhub | EU/US quotes, ETF data | `FINNHUB_API_KEY` env var | HIGH |
| brapi.dev | B3, FIIs, Brazilian equities | None (free tier) | HIGH |
| Bundesbank SDMX | Basiszins for Vorabpauschale | None | HIGH |
| OpenFIGI | ISIN → ticker mapping | None (rate-limited) | HIGH |

### Reference Doc Architecture

```
finyx/references/
  germany/
    tax-investment.md          # Abgeltungssteuer, Sparerpauschbetrag, Vorabpauschale
    tax-rules.md               # Real estate tax rules (from IMMO)
    pension.md                 # Riester/Rürup/bAV
    brokers.md                 # DE broker comparison
  brazil/
    tax-investment.md          # IR, DARF, FII exemptions
    pension.md                 # PGBL/VGBL/INSS
    brokers.md                 # BR broker comparison
  insights/                    # NEW for v1.1
    scoring-methodology.md     # Health score rubric + benchmarks
    projection-formulas.md     # FV formulas + return rate assumptions
  disclaimer.md
```

---

## What NOT to Add

| Option | Why not |
|--------|---------|
| `financejs` npm package | Two formulas don't justify a dependency; Claude computes them inline |
| `cli-progress`, `progress`, or any bar-chart npm library | Requires runtime; Unicode characters in prompt output achieve the same result |
| Chart.js / terminal-kit / blessed | Web/full-TUI frameworks — wrong layer entirely for a prompt-based system |
| SQLite or any database | Profile + cache are small JSON files; DB adds installation complexity for no benefit |
| `lodash` or data manipulation libraries | JSON aggregation is trivial at this scale; Claude handles it |
| Any web framework | Hard no — CLI-first, slash-command architecture |
| Yahoo Finance / Alpha Vantage | Established in v1.0 stack as explicitly excluded |

---

## Data Flow for `/fin:insights`

```
profile.json          → income, goals, risk tolerance, liquid assets
insights-cache.json   → structured numbers from previous advisor runs
scoring-methodology.md → rubric + benchmarks
projection-formulas.md → FV formulas + return rate lookup

Claude aggregates all four → generates:
  1. Income allocation bar chart (Unicode)
  2. Financial health score (1-10 per dimension)
  3. Net worth projection (FV calculation inline)
  4. Goal pace tracking (months-to-target arithmetic)
  5. Ranked recommendations (sorted by estimated EUR impact)
```

---

## Sources

- Finance.js (zero-dependency JS financial library): https://financejs.org/
- Bloomberg WealthScore methodology (1-10 scoring, 7 benchmarks): https://www.bloomberg.com/wealthscore-financial-health-calculator
- Financial Health Network — Spend/Save/Borrow/Plan pillars: https://finhealthnetwork.org/tools/financial-health-score/
- CFPB financial well-being scoring: https://www.consumerfinance.gov/consumer-tools/educator-tools/financial-well-being-resources/measure-and-score/
- 50/30/20 rule benchmark (2025 feasibility): https://247wallst.com/personal-finance/2025/02/15/is-the-50-30-20-budget-rule-still-even-possible-in-2025/
- Unicode sparkline characters (U+2581–U+2588): https://rosettacode.org/wiki/Sparkline_in_unicode
- Anthropic financial services skills pattern: https://github.com/anthropics/financial-services-plugins
- Existing v1.0 STACK.md findings (Finnhub, brapi.dev, Bundesbank SDMX, OpenFIGI) — all HIGH confidence, verified in prior research round

---
*Stack research for: Finyx v1.1 Financial Insights Dashboard*
*Researched: 2026-04-06*
