# Pitfalls Research

**Domain:** Financial insights/dashboard command for CLI personal finance advisor
**Researched:** 2026-04-06
**Confidence:** HIGH

---

## Critical Pitfalls

### Pitfall 1: Projection Precision Theater

**What goes wrong:**
Net worth projections and goal-pace calculations display exact numbers ("you will reach €847,312 in 6.4 years") that imply mathematical certainty. Users anchor to these numbers, make decisions based on them, and blame the tool when reality diverges.

**Why it happens:**
It is easy to compute a deterministic compound-growth formula and present the output with full precision. The number is technically correct given assumptions, but the assumptions carry enormous uncertainty that the output obscures.

**How to avoid:**
- Present projections as ranges, not point estimates ("€680k–€1.1M in 6–9 years based on 0–4% annual growth")
- Surface every assumption used in the projection inline, not in a footnote
- Add explicit scenario labels: conservative / base / optimistic
- Never display cents in long-range projections — round to nearest €1k to signal uncertainty
- Read growth assumptions from `profile.json assumptions.appreciation` — do not hardcode

**Warning signs:**
- Output shows "€X,XXX.XX in exactly Y years"
- Projection section has no stated assumptions
- Single growth rate hardcoded in the prompt instead of read from profile

**Phase to address:**
Net worth projection feature — before any projection output is drafted

---

### Pitfall 2: Stale Data Synthesized into Confident Recommendations

**What goes wrong:**
The `/fin:insights` command aggregates outputs from tax, invest, pension, and broker advisors. If those advisors have never been run, the insights command synthesizes a "complete picture" from partial or missing data, presenting confident recommendations based on gaps. There is no way to distinguish "user has no investments" from "user never ran /finyx:invest".

**Why it happens:**
The existing specialist commands are conversational — they write nothing to disk except what the user explicitly saves. The only durable source is `profile.json`, which may have empty sections for holdings, pension contributions, or broker fees.

**How to avoid:**
- Add a data-completeness check in Phase 1 of insights: verify which profile sections are populated (`countries.germany.brokers[].holdings`, pension fields, etc.)
- Report coverage confidence per section: "Investment allocation: PARTIAL — no holdings recorded. Run /finyx:invest to complete."
- Never generate a ranked recommendation for a domain where the profile data is absent
- Define minimum viable data thresholds — what fields must exist before each insight type fires

**Warning signs:**
- Insights output ranks recommendations for pension or investments without any holdings/contribution data in profile.json
- The command has no Phase 1 data-completeness gate
- "Top recommendations" section appears regardless of what data is present

**Phase to address:**
Data-completeness gate — first thing built in the insights command, before any analysis logic

---

### Pitfall 3: Tax Efficiency Score Misleads Across Jurisdictions

**What goes wrong:**
A unified "tax efficiency score" (e.g., 67/100) aggregates German and Brazilian tax optimization metrics into one number. A user maximizing Sparerpauschbetrag but ignoring DARF deadlines gets a misleadingly high score. Worse, optimizations valid in Germany may be irrelevant or wrong in Brazil.

**Why it happens:**
Synthesis feels like a feature — one number is cleaner than two. But DE and BR tax rules have different allowances, different timing, different penalty structures, and different legal risk profiles. Combining them conflates incompatible metrics.

**How to avoid:**
- Score tax efficiency per country, never as a combined metric
- Label each score explicitly: "Germany tax efficiency: 4/5" and "Brazil tax efficiency: 2/5"
- Cross-border insight should explain interactions (DBA, withholding credits), not produce a merged score
- Ensure all tax efficiency logic references the same reference docs used by `/finyx:tax` — do not duplicate tax rules inline in the insights prompt

**Warning signs:**
- A single numeric score labeled "overall tax efficiency"
- Tax optimization recommendations that do not specify which country's rules apply
- Tax logic duplicated in the insights prompt rather than `@` included from reference docs

**Phase to address:**
Tax efficiency scoring — during prompt design, before first draft of scoring rubric

---

### Pitfall 4: Income Allocation Benchmarks Applied Without Country Context

**What goes wrong:**
The 50/30/20 rule is applied universally. In Germany at €210k gross, effective tax plus social contributions consume ~45% of income before any discretionary spending. Telling a user their "savings rate is too low" when they allocate 15% to investments on top of mandatory bAV and RV contributions produces a misleading output.

**Why it happens:**
Allocation benchmarks are imported from generic personal finance content without adjusting for tax burden, mandatory deductions, or social security structures that vary by country.

**How to avoid:**
- Compute allocation analysis on net-of-tax, net-of-mandatory-deductions income, not gross
- Surface the actual net income denominator used, explicitly
- Apply German/Brazilian context: RV contributions, bAV contributions, and Krankenversicherung are not voluntary savings
- Label benchmarks as country-adjusted: "Adjusted voluntary savings rate (excl. mandatory pension): X%"
- Do not use US-centric benchmarks for German/Brazilian users

**Warning signs:**
- Allocation analysis uses `gross_income` as the denominator without netting mandatory deductions
- Benchmark comparisons cite generic "financial advisor" targets without country qualifier
- Monthly savings rate appears lower than it actually is because mandatory contributions are excluded

**Phase to address:**
Income allocation analysis — during formula design, before the first benchmark table is drafted

---

### Pitfall 5: Scope Creep into Expense Tracking

**What goes wrong:**
The insights command starts asking users to input monthly spending by category to compute a needs/wants/savings breakdown. This turns a one-shot advisory report into a recurring data-entry workflow, changing the product from "advisor" to "budget tracker." This directly conflicts with the explicit out-of-scope decision: "Budget tracking / expense categorization — different product, dilutes focus."

**Why it happens:**
Allocation analysis naturally surfaces the question "but we need spending data." The temptation is to collect it inline. One prompt question becomes a habit, which becomes a required data field.

**How to avoid:**
- Derive allocation estimates from `profile.json` only: gross income, monthly commitments, known investment contributions
- Where spending data is unknown, state the gap explicitly and give a methodology the user can apply manually
- Never persist spending categories to `profile.json`
- If the user volunteers spending data in a session, use it for that session only — do not prompt for it or save it

**Warning signs:**
- Insights prompt asks "how much do you spend on food/rent/entertainment?"
- A `spending` or `budget` section appears in `profile.json` as a result of running insights
- Phase 2 of the insights command contains category-by-category data collection

**Phase to address:**
Feature scoping — during command specification, before any data-collection phase is written

---

### Pitfall 6: Recommendation Ranking Without Impact Quantification

**What goes wrong:**
"Top recommendations" are ranked by perceived importance or domain coverage rather than financial impact. The user sees "maximize Sparerpauschbetrag" listed above "increase bAV contribution to the tax-free limit" when the latter might save €3,000/year vs €200/year.

**Why it happens:**
Calculating financial impact requires knowing tax rates, current allocations, and rule thresholds — it is harder than generating a qualitative list. Qualitative ranking ships faster.

**How to avoid:**
- Every ranked recommendation must include an estimated annual impact in euros
- Impact must be computed from actual profile data (marginal rate, current allocations, known allowances) not generic averages
- Flag estimates: "~€X/year (estimate — based on marginal rate of 44.31% and current Sparerpauschbetrag usage)"
- Rank by impact descending; tie-break by effort (low-effort first)
- Include a confidence qualifier: HIGH (calculated from profile data), MEDIUM (estimated from partial data), LOW (structural/qualitative)

**Warning signs:**
- Recommendations section contains no euro amounts
- Ranking is alphabetical, category-ordered, or domain-ordered rather than impact-ordered
- All recommendations have equal visual weight with no differentiation by size of opportunity

**Phase to address:**
Recommendation engine — during prompt design for the recommendations section

---

### Pitfall 7: Legal Disclaimer Regression

**What goes wrong:**
The `/fin:insights` command generates a synthesized "financial health report" with scores, rankings, and projections. This consolidated format makes it easy to omit the disclaimer coverage that individual commands have. A report without inline disclaimers exposes the project to misuse claims regardless of any buried footer.

**Why it happens:**
`@~/.claude/finyx/references/disclaimer.md` is included in individual commands, but an insights command might not inherit it, or might only include it once at the bottom after generating authoritative-sounding content throughout.

**How to avoid:**
- Include `@~/.claude/finyx/references/disclaimer.md` in `<execution_context>` of insights — same as all other commands
- Add inline disclaimer anchors at any section producing projections, scores, or ranked recommendations: "This is an estimate, not financial advice."
- The command must not produce text using "you should", "you must", or "you need to" without the advisory qualifier

**Warning signs:**
- `<execution_context>` block does not reference `disclaimer.md`
- Disclaimer appears only at the end of the output, after projections and scores
- Output uses directive language without advisory qualifiers

**Phase to address:**
Command skeleton — disclaimer inclusion verified before any output section is written

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Duplicate tax logic in insights prompt instead of `@` including reference docs | Faster to write one self-contained prompt | Rules get out of sync when reference docs are updated; two sources of truth diverge silently | Never |
| Hardcode growth/inflation assumptions in prompt | Simple first implementation | Diverges from `profile.json assumptions`; impossible to scenario-test | Never — read from profile |
| Generate insights without data-completeness check | Works when profile is fully populated | Confident recommendations from zero data; erodes trust permanently | Never |
| Single combined output section mixing projections + recommendations + scores | One-pass generation | User cannot consume output; impossible to add/remove sections later | Never |
| Collect spending data interactively to fill allocation gaps | More accurate allocation breakdown | Scope creep into budget tracking; breaks advisory-only model | Never |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| `/finyx:tax` reference docs | Re-implement tax rules inline in insights prompt | `@` include the same tax reference docs used by `/finyx:tax`; never duplicate |
| `profile.json` | Read only top-level fields, miss nested structure | Read `countries.germany.brokers[].holdings[]`, pension sub-objects — profile is deeply nested |
| `/finyx:pension` output | Assume pension data is in profile.json | Pension command is conversational; only fields the user explicitly saved are in profile — verify what is actually persisted |
| Cross-border profile | Apply Germany logic to all users | Gate on `identity.cross_border` and `identity.income_countries` before applying country-specific analysis |
| Tax year staleness | Skip staleness check because insights "just aggregates" | Include the same staleness check that `/finyx:tax` runs — insights inherits the same staleness risk |
| Goal pace tracking | Calculate pace from gross income and goal amount only | Must factor in current savings rate, existing assets, investment returns, inflation — bare division is misleading |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Logging `profile.json` contents to terminal output | Income figures, marginal rates exposed in terminal history | Never echo raw profile.json; extract only what is needed for display |
| Including CPF or tax identification numbers in report output | PII in Markdown output files that may be committed or shared | Redact or omit identity numbers from all generated report text |
| Saving user-volunteered session data to `profile.json` without explicit consent | User assumes data is ephemeral; it persists | Only write to `profile.json` when user explicitly confirms; show what will be written before writing |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Report too long to be actionable | User reads first section, skips the rest; misses critical recommendations | Lead with "Top 3 actions this month" before detailed analysis |
| Score without context ("Tax efficiency: 3/5") | User does not know if 3/5 is good, bad, or what drove the score | Always show the score with contributing factors and the gap to close |
| Projections without stated horizon | "You will reach your goal" — when? | Always state projection horizon explicitly: "at current rate, by 2034 (8 years)" |
| Recommendations for domains with no data | User gets advice for situations they have not configured yet | Gate recommendations on whether the relevant country profile section is populated |
| Treating incomplete profile as complete | Silent assumptions fill gaps; user does not know what is missing | Always emit a coverage report at the top: which sections are complete, which are missing |

---

## "Looks Done But Isn't" Checklist

- [ ] **Data completeness gate:** Verify all required profile sections are populated before generating each insight type — not just that `profile.json` exists
- [ ] **Disclaimer inclusion:** Confirm `disclaimer.md` is in `<execution_context>` AND inline qualifiers appear at every projection/score/recommendation section
- [ ] **Country gating:** DE-specific insights do not fire where `countries.germany.tax_class` is null; BR-specific insights do not fire where `countries.brazil.gross_income = 0`
- [ ] **Tax year staleness check:** The same staleness check from `/finyx:tax` is present — not skipped because "insights just reads profile data"
- [ ] **Impact quantification:** Every ranked recommendation has an estimated annual euro impact, not just a qualitative label
- [ ] **Projection assumptions visible:** Every forward projection shows which `profile.json` fields were used as inputs
- [ ] **No spending data collection:** No phase in the command prompts for or persists expense/budget categories

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Projections without assumptions surfaced | LOW | Add assumption table before each projection; re-test output |
| Stale data synthesized into confident recommendations | LOW | Add data-completeness check in Phase 1; add coverage report to output header |
| Tax logic duplicated in insights prompt | MEDIUM | Extract duplicated rules, replace with `@` includes of existing reference docs; audit for drift |
| Scope creep: spending data collection added | MEDIUM | Remove data-collection phase; replace with gap statements; decide whether any captured data should be pruned from profile.json |
| Disclaimer omitted from aggregated output | LOW | Add to `<execution_context>` and add inline anchors; review all directive-sounding sentences |
| Unified cross-country tax score | LOW | Split into per-country scores; remove aggregated metric; update section headers |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Projection precision theater | Net worth projection design | Output shows ranges plus assumptions, never point estimates without uncertainty |
| Stale data synthesis | Phase 1 data-completeness gate | Insights run on empty profile produces coverage report, not fabricated recommendations |
| Cross-jurisdiction tax score | Tax efficiency scoring design | Output shows per-country scores; no combined metric exists |
| Gross-income allocation benchmarks | Income allocation formula design | Denominator confirmed as net-of-mandatory-deductions; assumptions visible |
| Scope creep into tracking | Command specification / feature scoping | No prompt phase collects or persists spending categories |
| Unquantified recommendation ranking | Recommendation engine design | Every recommendation includes estimated euro impact |
| Disclaimer regression | Command skeleton (first commit) | `disclaimer.md` in `<execution_context>`; inline qualifiers at projection/score/recommendation sections |

---

## Sources

- Project constraints and out-of-scope decisions: `.planning/PROJECT.md`
- Existing command patterns (disclaimer inclusion, staleness check): `commands/finyx/tax.md`, `commands/finyx/invest.md`
- Actual profile schema and field structure: `.finyx/profile.json`
- Pre-existing pitfalls research (v1.0 baseline): prior `.planning/research/PITFALLS.md` (preserved below)

---

## Prior Pitfalls (v1.0 Baseline — Preserved)

The following pitfalls were identified during v1.0 research and remain applicable to the full system.

### Stale Tax Rules Presented as Current

Tax rules change annually. Reference docs must include `tax_year:` and `valid_until:` metadata. Vorabpauschale Basiszins changes every year via BMF circular. Brazilian Law 15,270/2025 (effective 2026-01-01) introduced dividend withholding and new exemption thresholds. Any reference doc written before late 2025 is missing these.

**Phase:** Foundation / German and Brazilian Tax Advisor phases.

### Wrong Jurisdiction Applied to Cross-Border Users

Jurisdiction inferred from a single profile field misses dual obligations. Brazilian expats in Germany owe Brazilian DARF on capital gains from Brazilian stocks regardless of German residency. Profile must capture: primary tax residency, secondary obligations, expat status.

**Phase:** User Financial Profile phase — before any tax advisor phase.

### Confidence Signaling Without Certainty

LLMs default to confident prose. Without explicit prompt instructions, outputs omit hedging even for complex calculated values like Vorabpauschale and come-cotas. Every advisor agent must distinguish "orientation advice" from "estimates" and flag estimates explicitly.

**Phase:** All advisor phases — establish in first agent, enforce as template constraint.

### Freistellungsauftrag Split Across Multiple Brokers

Advisor calculates each account's Sparerpauschbetrag exposure in isolation. Multi-broker split must be aggregated. Total across all brokers cannot exceed €1,000 (single) / €2,000 (married). German tax advisor must include a mandatory multi-broker check step.

**Phase:** German Tax Advisor phase.

### Riester/Rürup Net Benefit Oversimplification

100% deductibility looks attractive but payouts are fully taxed at retirement. Pension advisor must calculate net benefit (contribution tax savings minus expected retirement tax cost) before recommending these products.

**Phase:** German Pension Planning phase.

### Brazilian FII Exemption Conditions Ignored

FII dividends are exempt only if: at least 100 quota holders, and the individual holds less than 10% of total quotas and generates less than 10% of fund income. Reference docs must include the full eligibility check, not just the headline rate.

**Phase:** Brazilian Tax Advisor and Investment Advisor phases.

### Come-Cotas Liquidity Surprise

Accumulating fixed-income and multimarket funds owe tax in May and November without any distribution. Users need liquidity to pay without forced unit sales. Brazilian tax advisor must surface this for all holders of qualifying funds.

**Phase:** Brazilian Tax Advisor phase.

### Vorabpauschale Basiszins Hardcoded

Do not hardcode the Basiszins in reference docs without a `valid_for_tax_year` marker. Add to annual tax checklist. Source: https://www.bundesfinanzministerium.de

**Phase:** German Tax Advisor phase.

### Prompt Injection via User Financial Data

Free-form profile fields embedded in prompts can carry injected instructions. Wrap all user-supplied fields in `<user_data>` delimiters. Use structured enums for high-trust fields. Never grant `--dangerously-skip-permissions`.

**Phase:** Shared Memory System / User Financial Profile phase.

### "Advisory Only" Eroded by Output Format

Outputs formatted as "Action Plan" or "Filing Instructions" read as directives. Use hedged language: "consider", "estimate", "this may apply". Standard disclaimer block required on every generated report.

**Phase:** All phases — establish in first agent.

### Brazil Tax Reform Lag (Law 15,270/2025)

Brazilian reference docs must reflect the 2026-01-01 rules: 10% dividend withholding tax on dividends >BRL 50k/month, new individual income tax exemption up to BRL 5,000/month, JCP rate increase (15% to 20%). Include `effective_from: 2026-01-01` version metadata.

**Phase:** Brazilian Tax Advisor phase.

### bAV Portability Risk Not Surfaced

bAV recommendation without portability warning. If user changes employers, transferring bAV can be complex. Flag that bAV is primarily valuable for long-tenure employees.

**Phase:** German Pension Planning phase.

---
*Pitfalls research for: financial insights/dashboard command added to existing CLI personal finance advisor*
*Researched: 2026-04-06*
