# Feature Landscape

**Domain:** Financial insights dashboard — `/fin:insights` command for Finyx personal finance CLI
**Researched:** 2026-04-06
**Confidence:** HIGH (financial health scoring patterns are well-established; CLI-specific adaptations are medium confidence based on domain reasoning)

---

## Feature Landscape

### Table Stakes (Users Expect These)

Features a "financial health" or "insights" command must have. Missing these = output feels like a status dump, not an advisor.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Income allocation snapshot | Users expect to see how their income is distributed (needs/savings/investments/debt) vs a benchmark like 50/30/20 | LOW | Read from `profile.json`; no new data required; compare to configurable benchmark |
| Tax efficiency summary | Show used vs unused tax allowances (Sparerpauschbetrag, PGBL 12% limit) with explicit gap and action | LOW | All data already in profile + tax advisor outputs; pure synthesis |
| Net worth snapshot | Current assets minus liabilities — the single most important financial health number | LOW | Profile already captures assets/liabilities; computation is arithmetic |
| Top-N actionable recommendations | Must end with a ranked list of concrete next steps, not just observations | LOW | Synthesis across all advisor domains; ranking by financial impact |
| Goal pace indicator | "At current savings rate, you reach €X target in Y months/years" | MEDIUM | Requires savings rate + target from profile; simple compound projection |
| Emergency fund check | 3–6 months expenses check is a universal table-stakes insight; users expect it | LOW | Profile has monthly expenses + liquid savings |
| Debt-to-income ratio | Benchmark against the <35% healthy threshold; flag if over | LOW | Profile already has debt obligations and income |

### Differentiators (Competitive Advantage)

Features that make `/fin:insights` meaningfully better than generic finance dashboards — enabled by Finyx's cross-domain knowledge and shared profile.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Cross-advisor intelligence synthesis | Surfaces interactions between domains: "pension gap means you're under-utilizing Rürup deduction which would also reduce your marginal tax rate on investments" — no siloed tool does this | HIGH | Must read outputs from tax + pension + invest advisors and reason across them |
| Country-aware benchmarks | 50/30/20 breaks down in Germany (52%+ mandatory deductions); use country-specific benchmarks based on net income, not gross | MEDIUM | German benchmark: 30% savings feasible on net, not gross. Brazilian benchmark: different due to previdência and cost of living |
| Tax optimization gap score | Express unused German/Brazilian tax allowances as a single "X% of potential tax savings unrealized" metric | MEDIUM | Aggregate Sparerpauschbetrag unused + Rürup under-contribution + PGBL under-contribution into one score |
| Net worth forward projection | "At current rate: €X in 5 years, €Y in 10 years" — uses actual savings rate and risk-adjusted return from profile | MEDIUM | Not just current snapshot; show trajectory with current behavior vs optimized behavior |
| Portfolio-vs-pension gap warning | Compare total projected retirement income (statutory + private pension) against a replacement income target | HIGH | Requires pension planning data + investment data; unique because Finyx has both |
| DE/BR interaction flag | Warn if Brazilian PGBL deduction is not being declared in German Anlage AUS / vice versa when relevant for expats | HIGH | Cross-border advisory is core Finyx differentiator; insights must surface the cross-country tax interactions |
| Recommendation impact ranking | Each recommendation shows estimated annual financial impact in € (or BRL) — e.g., "Using full Sparerpauschbetrag saves ~€263/year at 26.375% Abgeltungssteuer" | MEDIUM | Makes prioritization concrete; forces advisor to quantify, not just flag |
| Year-end tax action list | Insight run in Q4 triggers "actions to take before Dec 31" section: harvest losses, top up Rürup, check Vorabpauschale cash position | MEDIUM | Date-aware output; very high value for annual planning context |

### Anti-Features (Commonly Requested, Often Problematic)

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Budget tracking / expense categorization | "Show me where my money goes" sounds like insights | Different product (transaction-level tracking); requires bank feed or manual entry; dilutes the advisory focus | Acknowledge the gap; direct to YNAB/MoneyMoney; insights work from declared income allocation, not transaction history |
| Historical trend charts / graphs | "Show me my net worth over time" is a natural ask | CLI has no charting capability; storing historical snapshots adds state management complexity; v1.1 is a point-in-time advisor | Give a projected forward trend instead; historical tracking is a v2 feature after persistent state is validated |
| Financial health score as a single number | Score feels objective and gameable (users want to "hit 100") | A single number hides the composition; users optimize the score, not their finances; creates false precision | Show component ratings (emergency fund: GOOD, savings rate: NEEDS IMPROVEMENT) without aggregating to one number |
| Comparison to other users / benchmarks by age | "Am I doing better than average for my age?" is compelling | Demographic benchmarking requires maintaining population data; DE/BR datasets differ; creates envy, not action | Compare to standards (e.g., "3–6 months emergency fund") not to "people like you"; avoids data maintenance burden |
| Spending advice / frugality tips | Generic "cut your coffee" advice feels like insights | Patronizing for a user who hasn't declared spending; Finyx has no transaction data to base this on | Only recommend reducing a category if profile data explicitly shows it above benchmark; never speculate |
| Insurance recommendations | Insurance is part of financial health; users will expect it | Explicitly deferred to v2 in PROJECT.md; data sourcing is unsolved | Flag insurance as "not assessed — run `/fin:insurance` when available" |
| Auto-refresh / scheduled runs | "Run weekly insights automatically" sounds useful | Requires a daemon/scheduler; CLI tool, not a service; outside architecture constraints | Users run `/fin:insights` on demand; document recommended cadence (monthly, or before major decisions) |

---

## Feature Dependencies

```
/fin:insights (synthesis command)
    └──reads──> .finyx/profile.json (income, goals, assets, liabilities, emergency fund, debt)
    └──reads──> last tax advisor output or re-derives from profile (tax allowances used/available)
    └──reads──> last investment advisor output or re-derives (portfolio allocation, risk profile)
    └──reads──> last pension advisor output or re-derives (pension gap estimate, contribution levels)

Income Allocation Analysis
    └──requires──> net income from profile.json
    └──requires──> expense breakdown from profile.json (housing, savings %, debt payments)

Tax Efficiency Score
    └──requires──> German: Sparerpauschbetrag used/limit, Rürup contribution vs max, Vorabpauschale status
    └──requires──> Brazilian: PGBL contribution vs 12% limit, DARF compliance flag
    └──requires──> Both from profile.json (present without running individual advisors)

Net Worth Snapshot + Projection
    └──requires──> Assets list from profile.json
    └──requires──> Liabilities list from profile.json
    └──requires──> Monthly savings rate from profile.json
    └──enhances with──> Expected return assumption (from investment advisor risk profile)

Goal Pace Tracking
    └──requires──> Financial goals with target amount + date from profile.json
    └──requires──> Current monthly savings rate
    └──depends on──> Net worth snapshot (baseline)

Cross-Advisor Intelligence
    └──requires──> All domain outputs above computed first
    └──surfaces──> Interactions: pension gap → under-utilized Rürup → higher tax bill on investments
    └──surfaces──> DE/BR cross-border: PGBL foreign income declaration in German Anlage AUS

Recommendation Ranking
    └──requires──> All analysis sections completed
    └──requires──> Impact quantification in € or BRL
    └──outputs──> Ranked list sorted by estimated annual financial impact
```

### Dependency Notes

- **All features require `profile.json` to be populated:** `/fin:insights` must gate on profile completeness. If profile is missing key fields (income, goals, assets), prompt user to run `/fin:profile` first.
- **Tax efficiency is derivable from profile alone:** The tax advisor outputs are helpful context but insights can re-derive allowance gaps directly from profile data — no hard dependency on prior advisor runs.
- **Cross-advisor intelligence is the hardest feature:** It requires reasoning across 3+ domains simultaneously. Implemented as a late synthesis step after all individual analyses are computed in the same command run.
- **Projection accuracy depends on assumptions:** Net worth forward projection uses declared savings rate + risk-adjusted return from profile. Must show assumptions inline so users can validate, not treat as oracle output.

---

## MVP Definition

### Launch With (v1.1)

Minimum viable `/fin:insights` command that delivers real value.

- [ ] Income allocation vs benchmark — read net income + expense breakdown from profile; compare to 50/30/20 adjusted for German net income reality
- [ ] Tax efficiency gaps — unused Sparerpauschbetrag, under-utilized Rürup, PGBL under-contribution; show in € terms
- [ ] Net worth snapshot — current assets minus liabilities from profile; no projection yet
- [ ] Emergency fund check — months covered vs 3–6 target
- [ ] Goal pace indicator — for each declared goal: "on track / off track / months to target at current rate"
- [ ] Top-5 ranked recommendations — each with estimated annual financial impact in € or BRL
- [ ] Cross-advisor intelligence — at least flag the most impactful cross-domain interaction detected

### Add After Validation (v1.x)

- [ ] Net worth forward projection (5-year + 10-year) — needs validated savings rate data from real user runs
- [ ] Year-end tax action list — date-aware section; add when seasonal utility is confirmed
- [ ] Recommendation impact quantification refinement — initial estimates will be rough; calibrate from user feedback
- [ ] DE/BR cross-border interaction detection — complex logic; validate single-country flow first

### Future Consideration (v2+)

- [ ] Historical net worth tracking — requires persistent state beyond current profile.json
- [ ] Portfolio-vs-pension gap warning — needs pension projection model to be solid
- [ ] Insurance gap detection — blocked on insurance advisor (v2 scope per PROJECT.md)

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Income allocation vs benchmark | HIGH | LOW | P1 |
| Tax efficiency gaps (DE + BR) | HIGH | LOW | P1 |
| Net worth snapshot | HIGH | LOW | P1 |
| Emergency fund check | HIGH | LOW | P1 |
| Goal pace indicator | HIGH | MEDIUM | P1 |
| Top-N ranked recommendations | HIGH | MEDIUM | P1 |
| Cross-advisor intelligence (basic) | HIGH | HIGH | P1 |
| Net worth forward projection | HIGH | MEDIUM | P2 |
| Year-end tax action list | MEDIUM | MEDIUM | P2 |
| Recommendation impact in € | MEDIUM | MEDIUM | P2 |
| DE/BR cross-border interactions | MEDIUM | HIGH | P2 |
| Portfolio-vs-pension gap | HIGH | HIGH | P3 |

**Priority key:**
- P1: Must have for v1.1 launch
- P2: Should have, add in v1.1 or v1.2
- P3: Future milestone

---

## Competitor Feature Analysis

| Feature | Mint / YNAB | Finanzguru (DE) | PortfolioPilot | Our Approach |
|---------|-------------|-----------------|----------------|--------------|
| Income allocation | Transaction-derived, automatic | Transaction-derived via bank sync | Not applicable | Declared income + manual allocation from profile; no bank sync needed |
| Tax efficiency | None (US-only Mint) | German tax tips (generic) | Portfolio-level only | Country-specific gap calculation with exact € amounts |
| Net worth | Tracked over time via account sync | Tracked via bank sync | Investment accounts only | Point-in-time from profile; forward projection without sync |
| Cross-domain advice | None | None | None | Core differentiator — pension gap → tax gap → investment gap |
| Goal pacing | Basic (savings goal trackers) | None | None | Profile-declared goals with projection |
| CLI / no sync | None (UI-only, requires sync) | None (UI-only, requires sync) | None | Privacy-first; no external data required; all derived from profile |

---

## Sources

- [Financial Health Pulse 2025 U.S. Trends Report — Financial Health Network](https://finhealthnetwork.org/research/financial-health-pulse-2025-u-s-trends-report/)
- [Top Personal Finance Metrics 2025 — The Gild Group](https://thegildgroup.com/financial-health-metrics/)
- [Financial Health Score components — FasterCapital](https://fastercapital.com/content/Harnessing-the-Power-of-the-Financial-Health-Score.html)
- [50/30/20 Rule — Chase](https://www.chase.com/personal/banking/education/budgeting-saving/50-20-30-budget-rule)
- [8 Personal Finance Ratios — U.S. News](https://money.usnews.com/money/personal-finance/family-finance/articles/personal-finance-ratios-to-know-at-all-times)
- [Net Worth Projection — ProjectionLab](https://projectionlab.com/net-worth)
- [AI Financial Advisor cross-domain patterns — useorigin.com](https://useorigin.com/resources/blog/technical-overview)
- [AI Financial Advisors with built-in tax features — Oreate AI](https://www.oreateai.com/blog/ai-financial-advisors-with-builtin-tax-features/77c9bbfb2ad31e873e507e3bcb1a0edb)
- [Financial Dashboard Customization for Advisors 2025 — RightCapital](https://www.rightcapital.com/blog/financial-dashboard/)

---
*Feature research for: Finyx /fin:insights command*
*Researched: 2026-04-06*
