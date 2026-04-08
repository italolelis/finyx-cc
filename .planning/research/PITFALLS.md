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

---

---

# v1.2 Health Insurance Advisor Pitfalls

**Domain:** PKV vs GKV decision advisor for Germany
**Researched:** 2026-04-06
**Confidence:** HIGH

---

## Critical Pitfalls

### Pitfall 1: PKV Premium Projection Ignores Compounding Age-Based Increases

**What goes wrong:**
The tool projects PKV costs using the current premium extrapolated linearly, or applies a flat annual increase rate. In reality, PKV premiums grow non-linearly because risk cost increases with age, Altersrückstellungen (age reserves) only partially offset this, and medical inflation has consistently outpaced general inflation. Projecting €400/month today as "€600 by retirement" understates the realistic range by 2-4x.

**Why it happens:**
Linear extrapolation is the simplest implementation. Actual actuarial tables are tariff-specific and proprietary. Developers substitute a flat 3-4% growth assumption without surfacing that this is a floor, not a ceiling.

**Consequences:**
User switches to PKV based on a favorable 20-year projection, then faces retirement premiums 2-3x what was modeled. GKV return is blocked after 55 (see Pitfall 4). No recovery path.

**Prevention:**
- Never present a single-line PKV projection. Always show conservative / base / optimistic scenarios
- Conservative scenario: 6-8% annual premium growth (historical average for many tariffs in recent years)
- Base scenario: 4-5% annual growth
- State explicitly: "PKV premium growth is not guaranteed. Historical increases 2020-2026 were substantial due to hospital cost reform and medication prices"
- Flag that Altersrückstellungen reduce but do not eliminate age-based increases
- Cite the 2026 PKV premium increases driven by Krankenhausreform and medication costs as a recent calibration point

**Detection:**
- Output shows a single monthly premium for retirement age
- No scenario range in the long-term projection
- Growth rate is hardcoded rather than labeled as an assumption

**Phase to address:** PKV cost projection engine — before any projection output is drafted. Verify scenarios exist and growth assumption is labeled.

---

### Pitfall 2: Familienversicherung Cost Ignored — Family PKV Cost Severely Underestimated

**What goes wrong:**
The tool calculates PKV cost as the individual's monthly premium only. In GKV, a non-working spouse and children are covered at no additional cost via Familienversicherung (§10 SGB V). In PKV, every family member requires a fully separate policy with individual underwriting and premiums. A family with two children and a non-working spouse could pay €600-1,200/month in additional PKV premiums that GKV would provide for free.

**Why it happens:**
Profile has income fields for the primary user. The comparison agent calculates the employee's PKV premium and compares it to the employee's GKV contribution, omitting the family dimension entirely.

**Consequences:**
Families systematically underestimate PKV cost. The comparison shows PKV as cheaper or comparable when it is materially more expensive once family coverage is included. Irreversible switch made on incomplete data.

**Prevention:**
- Phase 1 of the comparison must check `profile.json` for: marital status, partner employment status, number of dependent children
- If partner is not employed or earns below GKV Geringfügigkeitsgrenze (€538/month in 2025): add estimated partner PKV premium to the PKV side of the comparison
- Per dependent child: add €100-200/month estimated PKV premium (flag as estimate, varies by tariff and health status)
- GKV side: partner and children covered at zero marginal cost via Familienversicherung — state this explicitly
- For a single-income family with 2+ children, output must include a warning: "GKV family coverage advantage is likely decisive for your situation"

**Detection:**
- PKV cost calculation only includes primary policyholder's premium
- Profile has `family.children > 0` or `family.partner_employed = false` but output shows no family premium line
- Comparison table has no "family coverage" row

**Phase to address:** Profile integration phase — family fields must be read before any cost comparison is generated.

---

### Pitfall 3: Terminology Confusion — Versicherungspflichtgrenze vs Beitragsbemessungsgrenze

**What goes wrong:**
Two separate thresholds are conflated:
- **Versicherungspflichtgrenze (JAEG / Jahresarbeitsentgeltgrenze):** The income threshold above which an employee is no longer obligated to be in GKV and may switch to PKV. In 2025: €73,800 gross/year. In 2026: ~€77,400 gross/year.
- **Beitragsbemessungsgrenze (BBG):** The cap on income used to calculate GKV contributions. In 2025: €62,100/year (monthly €5,175). Contributions are only assessed up to this ceiling.

Confusing these produces wrong eligibility decisions ("you earn above the threshold" when income is between BBG and JAEG) or incorrect GKV cost calculations (applying the contribution rate to full income above BBG).

**Why it happens:**
Both thresholds are income-based and change annually. Generic content online uses them interchangeably. A tool built from web-searched content frequently inherits this confusion.

**Consequences:**
User told they can switch to PKV when they cannot (income between BBG and JAEG but below JAEG), or GKV contribution calculated as too high by applying rate to income above BBG.

**Prevention:**
- German health insurance reference doc must define both thresholds separately with current values and update cadence
- Eligibility check: use only JAEG to gate whether PKV switching is permissible. Include the 3-consecutive-years rule for employees
- GKV cost calculation: cap GKV-assessable income at BBG before applying Beitragssatz
- Both thresholds must have `valid_for: [year]` metadata — they are adjusted annually (typically announced in October/November for the following year)
- Reference doc source: GKV-Spitzenverband Rechengrößen announcement, updated annually

**Detection:**
- Reference doc uses "threshold" without specifying JAEG vs BBG
- GKV contribution calculated on gross income above €5,175/month without a BBG cap
- Eligibility gate checks a single threshold for both "can you switch" and "how much do you pay"

**Phase to address:** Reference doc authoring — before any cost calculation logic is written. Both thresholds must be defined, sourced, and version-tagged.

---

### Pitfall 4: GKV Return Lock-In After 55 Not Prominently Surfaced

**What goes wrong:**
The tool presents PKV as reversible ("you can always go back to GKV if needed") or buries the return restriction in a footnote. Under §6 Abs. 3a SGB V, employees who have been voluntarily PKV-insured for more than 5 years and are 55+ become permanently exempt from mandatory GKV insurance. They cannot re-enter GKV even if their income drops below the JAEG.

The only practical exit paths after 55 are:
1. Familienversicherung (own income below €538/month — effectively only through unemployment or retirement with minimal pension)
2. Basistárif (PKV's GKV-equivalent fallback — covers basic benefits at GKV-level cost but is stigmatized and restricted)
3. Civil servant track or major life event that re-establishes GKV eligibility

**Why it happens:**
The lock-in is a legal rule buried in SGB V. Advisory tools that rely on web-searched content often find the general "switching back is possible" framing without surfacing the age-based permanent exemption.

**Consequences:**
User switches to PKV at 35, income drops at 52 (illness, part-time, parental leave), and discovers they cannot return to the solidarity system. PKV premiums in reduced-income scenarios can consume 20-30% of net income.

**Prevention:**
- All PKV recommendations must include a prominently placed warning (not a footnote) for users currently under 55
- Warning text: "After age 55 with 5+ years in PKV: return to GKV is legally blocked in most circumstances. This decision is effectively permanent."
- Age-specific risk flag: if user is 45-54, flag as elevated urgency — the lock-in window is approaching
- Reference the §6 Abs. 3a SGB V provision with a source link
- The output must also surface the Basistárif as the legal safety net, with its limitations explained

**Detection:**
- Output says "you can switch back if needed" without age qualification
- No warning about the post-55 lock-in appears in the recommendation
- No reference to §6 Abs. 3a SGB V

**Phase to address:** Recommendation output phase — before the final recommendation section is drafted.

---

### Pitfall 5: Beihilfe for Beamte (Civil Servants) Treated as an Edge Case

**What goes wrong:**
The tool applies the standard employee PKV/GKV comparison to Beamte (civil servants). This produces a completely wrong analysis. Beamte receive Beihilfe — a government subsidy covering 50-80% of medical costs (higher rates for dependents and retirees). This changes the economics fundamentally: a Beamter only needs to insure the remaining 20-50%, resulting in PKV premiums of €200-450/month for full coverage that would cost a non-Beamter €600-900/month.

Furthermore:
- GKV for Beamte means the government pays no employer contribution — the Beamter bears the full GKV contribution
- Pauschale Beihilfe (available in Hamburg, Bremen, Brandenburg, Thüringen, Berlin, Baden-Württemberg) provides a 50% subsidy toward GKV instead
- PKV is structurally the correct default for most Beamte, but the analysis must account for their specific Beihilfesatz and federal state

**Why it happens:**
Beamte are a minority of the population. Tools built for the general case silently apply employee-track logic. The profile may not have a field for civil servant status.

**Consequences:**
Tool compares PKV as if the Beamter pays full premiums, producing a false GKV-favoring result. Or it recommends PKV without flagging that the tariff must be designed as a Restkostenversicherung complementing Beihilfe coverage.

**Prevention:**
- Profile must include `employment_type` with at least: `employee`, `self_employed`, `beamter`, `beamter_anwaerter`
- If `employment_type = beamter`: use a separate Beihilfe-aware cost model
  - Collect Beihilfesatz (50% base, 70% with dependents, 80% pensioners — varies by state and personal circumstances)
  - PKV cost = premium for Restkostenversicherung only (not full-cover tariff)
  - GKV cost = full contribution (no employer share) — compare against Pauschale Beihilfe if in applicable state
- Flag: "For Beamte, PKV is almost always financially superior due to Beihilfe. Ensure your tariff is structured as Restkosten coverage for the gap Beihilfe does not cover."
- Reference the 6 states offering Pauschale Beihilfe: Hamburg, Bremen, Brandenburg, Thüringen, Berlin, Baden-Württemberg

**Detection:**
- `employment_type` field absent from profile schema
- Beamte comparison uses standard employee PKV premium without Beihilfe reduction
- No Restkostenversicherung distinction in tariff guidance

**Phase to address:** Profile schema extension (before command is built) and reference doc authoring.

---

### Pitfall 6: Health Data Privacy — Collecting Pre-Existing Conditions Without Adequate Framing

**What goes wrong:**
To estimate PKV risk surcharges and exclusions, the tool asks users to disclose pre-existing conditions, chronic illnesses, and mental health history. This health data is:
- Highly sensitive under GDPR (Art. 9 special category data)
- Being stored in `profile.json`, a file that may be committed to git, shared, or accessible to other processes
- Used to infer insurability, potentially creating a record the user did not intend to create

**Why it happens:**
PKV underwriting genuinely requires health data to estimate realistic premiums. The temptation is to collect it systematically and persist it.

**Consequences:**
User's pre-existing conditions recorded in a plaintext JSON file in their project directory. If the file is committed or shared, this data leaks. If it is used to generate output text that is then saved as a report, health data appears in Markdown files.

**Prevention:**
- Health questionnaire responses must be session-only — never written to `profile.json` or any file
- State this explicitly before asking health questions: "This session is used for cost estimation only. Your answers will not be saved."
- Use abstract categories for output (e.g., "elevated PKV surcharge likely" rather than naming the condition)
- Reports must not include specific conditions by name — only their financial impact on estimated premiums
- Never include health fields in `profile.json` schema
- Add a prominent note to the command that CLI sessions are not encrypted and users should be aware of terminal logging

**Detection:**
- `profile.json` schema contains any `health`, `conditions`, `diagnoses`, or `medications` fields
- Generated report output contains named conditions or diagnoses
- Health questionnaire phase writes to `profile.json`

**Phase to address:** Command specification phase — before the health questionnaire is designed. Privacy boundary must be established as a non-negotiable constraint.

---

### Pitfall 7: Stale Tariff Data Presented as Current

**What goes wrong:**
The tool uses tariff data (provider names, premium ranges, benefit tiers) from training data or web-scraped content that is 1-3 years old. PKV tariffs change annually. Major changes occurred in 2024-2026 (hospital reform, medication pricing). Showing a user that "Provider X starts at €380/month" when current quotes are €480/month produces a misleading comparison.

**Why it happens:**
Tariff data is not available via an official free API. The PKV market has 40+ providers, each with dozens of tariffs. Training data reflects snapshots in time.

**Consequences:**
User budgets based on stale premiums. Actual quotes from providers are significantly different. Tool's credibility collapses at the first real quote.

**Prevention:**
- All tariff figures in the reference doc must be labeled as ranges/estimates, not specific prices: "PKV premiums for a healthy 35-year-old typically range €350-600/month depending on provider and coverage level"
- The PKV provider research agent must use live web search to find current indicative ranges — never rely on hardcoded figures
- Reference doc must include `valid_for: [year]` and note that tariffs change January 1 each year
- Output must instruct user to get personal quotes from providers or a qualified broker (Versicherungsmakler) before deciding
- Flag the 2026 premium increases as a calibration signal: many tariffs increased 10-20% effective January 2026

**Detection:**
- Reference doc contains specific monthly premium figures without a year tag
- Agent output presents specific premiums without attributing them to live search or noting they are estimates
- No "get a quote" instruction in the recommendation

**Phase to address:** Reference doc authoring and agent output template — before any tariff figures are included in the reference doc.

---

## Moderate Pitfalls

### Pitfall 8: Anwartschaft for Expats Overlooked

**What goes wrong:**
The tool models insurance scenarios for expats returning to Germany (or Germans going abroad) without surfacing the Anwartschaft option. A PKV policyholder leaving Germany can maintain an Anwartschaft (dormant contract) for a small monthly fee (~€20-80/month depending on insurer), preserving their entry age and health status for reactivation. Missing this means:
- The expat cancels their PKV and loses their age/health rating permanently
- On return, they re-enter PKV at their current (higher) age with a new health questionnaire

The tool must not recommend "cancel PKV before going abroad" without first surfacing Anwartschaft.

**Prevention:**
- Cross-border detection in profile (`identity.cross_border = true` or planned extended absence) triggers an Anwartschaft section
- State clearly: "During Anwartschaft you have no coverage — a separate international health insurance is required"
- Distinguish große Anwartschaft (preserves full tariff including Altersrückstellungen) from kleine Anwartschaft (preserves entry age only)
- GKV Anwartschaft (KVdR pre-insurance period recognition): ~€70/month, relevant if user wants credit toward Krankenversicherung der Rentner on return

**Phase to address:** Expat section of health reference doc.

---

### Pitfall 9: Self-Employed PKV Underestimates True Cost

**What goes wrong:**
Employees get an employer subsidy toward PKV premiums (Arbeitgeberzuschuss — 50% of premium up to the employer's GKV contribution cap, ~€421/month in 2025). Self-employed individuals receive no subsidy and pay the full PKV premium out of pocket. The tool calculates the same premium for both groups without netting the employer contribution differential.

Additionally, self-employed individuals in GKV pay the full contribution rate themselves (no employer share), making the GKV side more expensive for them too, but the differential calculation must be done correctly.

**Prevention:**
- Profile must have `employment_type` as `self_employed` vs `employee`
- For employees: PKV out-of-pocket = premium minus Arbeitgeberzuschuss (min(50% × premium, €421/month in 2025))
- For self-employed: PKV out-of-pocket = full premium; GKV cost = full contribution rate (no employer share)
- The Arbeitgeberzuschuss cap is annually adjusted — include in reference doc with `valid_for` metadata

**Phase to address:** Cost calculation phase of the comparison command.

---

### Pitfall 10: PKV Basisabsicherung Tax Deduction Overstated

**What goes wrong:**
PKV premiums are tax-deductible, but only the portion that covers Basisabsicherung (basic coverage equivalent to GKV benefits). A comprehensive PKV tariff includes many benefits beyond GKV equivalence (private room, Chefarztbehandlung, etc.). The deductible portion is typically 70-90% of the total premium depending on tariff, not 100%.

Presenting the full PKV premium as tax-deductible overstates the tax benefit and distorts the net cost comparison.

**Prevention:**
- Reference doc must distinguish the Basisabsicherungsanteil from the total premium
- Deductible portion: use 80% as a conservative default estimate when exact split is unknown; flag as estimate
- Note the §10 EStG Sonderausgaben cap: €1,900 (employees) / €2,800 (self-employed) for all health/nursing care insurance combined
- For high earners, the deduction is often already capped — the marginal deductibility of additional PKV premium may be zero

**Phase to address:** Tax implications section of the comparison output.

---

### Pitfall 11: Misleading Long-Term GKV Cost Projection

**What goes wrong:**
Long-term projections assume GKV contributions grow only with income. In reality:
- The average Zusatzbeitrag (supplementary contribution) has risen from 0.9% in 2015 to 1.7% in 2024 and is projected to continue increasing
- GKV structural funding problems (aging population, rising healthcare costs) mean contribution rates are forecast to increase meaningfully over the next 20 years
- For high earners, GKV cost is capped at the BBG — PKV may be cheaper in absolute terms at high income levels, but this changes if the BBG increases

**Prevention:**
- GKV projection must use a growth scenario for Beitragssatz, not a flat rate
- Show a range: "if Zusatzbeitrag increases from 1.7% to 2.5% by 2035 (plausible based on GKV-Spitzenverband projections), your GKV cost rises from X to Y"
- Do not present GKV as the "stable" option vs PKV — both systems have cost growth, with different drivers
- For users at or near the BBG cap: note that GKV cost is capped, which structurally benefits high earners in GKV

**Phase to address:** Long-term projection section of the comparison output.

---

## Minor Pitfalls

### Pitfall 12: Beitragsrückerstattung (No-Claim Discount) Overcounted

PKV tariffs often advertise Beitragsrückerstattung — a partial premium refund if no claims are submitted for a year (typically 1-3 months of premium returned). Tools that include this in "effective annual PKV cost" overstate PKV savings because:
- Users with a chronic condition or regular prescriptions will never receive this benefit
- The refund creates an incentive to delay care, which has real health consequences
- Over a 30-year projection, the compounding benefit of Beitragsrückerstattung is frequently overstated

**Prevention:** Show BRE as a scenario modifier ("if you claim no benefits: effective cost X; if you do claim: cost Y"), not as a baseline assumption.

### Pitfall 13: Selbstbeteiligung Not Factored Into True Cost

PKV tariffs with high Selbstbeteiligung (excess/deductible — e.g., €1,000/year) have lower premiums. Comparing the headline premium without noting the deductible level understates true annual cost for users who use healthcare regularly.

**Prevention:** Always include the Selbstbeteiligung level alongside premium in any cost comparison. Compute expected true cost = premium + expected out-of-pocket from deductible.

### Pitfall 14: Pflegeversicherung Not Included in Total Insurance Cost

Both PKV and GKV members pay Pflegeversicherung (long-term care insurance). In GKV: contribution calculated at ~3.4% of income (higher without children). In PKV: a separate private Pflegepflichtversicherung (PPV) policy is required. The cost of PPV is often omitted from PKV total cost comparisons.

**Prevention:** Add a PPV line to the total PKV cost calculation. For GKV users, include the Pflegebeitrag in the total GKV cost. Both must be comparable.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Profile schema extension | Missing `employment_type`, `family.children`, `family.partner_employed` fields | Add all required fields before building the command |
| Reference doc authoring | Versicherungspflichtgrenze vs Beitragsbemessungsgrenze conflation | Define both separately with current values and annual update process |
| Health questionnaire design | Health data persisted to `profile.json` | Session-only policy established before questionnaire is written |
| Cost comparison engine | Family coverage omitted; Beamte handled incorrectly | Separate logic paths for single/family and employee/Beamter/self-employed |
| Long-term projection | Linear PKV growth assumption; single-line output | Scenario ranges (conservative/base/optimistic) required; GKV growth modeled too |
| Recommendation output | GKV return lock-in after 55 not surfaced | Prominent warning required regardless of user's current age |
| Tariff provider research agent | Stale hardcoded premium figures | Agent must use live web search; all figures labeled as estimates requiring broker verification |
| Tax implications section | Full PKV premium treated as deductible; §10 EStG cap ignored | Apply Basisabsicherungsanteil fraction; check against Sonderausgaben cap from profile |
| Expat detection branch | Anwartschaft not mentioned before "cancel PKV" guidance | Cross-border flag in profile triggers mandatory Anwartschaft section |
| Legal disclaimers | Insurance-specific disclaimer weaker than tax disclaimer | Health insurance advisory carries the same legal sensitivity as tax advice — same disclaimer standards apply |

---

## Legal and Privacy Concerns

### Health Data is Special Category under GDPR

Art. 9 GDPR classifies health data as a special category requiring explicit consent and heightened protection. For an open-source CLI tool:
- No health data should be persisted to disk in any form
- The tool should not create a durable record linking a user to specific health conditions
- Output reports must not name conditions — only their estimated cost impact
- Terminal output is not encrypted; users should be warned about logging

### Advisory Disclaimer for Insurance Decisions

Insurance decisions in Germany require a licensed Versicherungsmakler or Versicherungsberater for binding advice. The tool must not present itself as a replacement. Required disclaimer elements:
- Advisory only — not a binding quote
- User must consult a qualified broker or use official provider calculators for actual premiums
- Tax deduction estimates should direct user to a Steuerberater for confirmation
- PKV switching decisions should include the recommendation to consult an independent broker (not a tied agent), since PKV brokers earn significant commissions that can bias advice

### Keine Versicherungsberatung ohne Zulassung

Under §34d GewO, providing insurance advice for compensation requires a license (Versicherungsvermittler-Zulassung). The tool must position itself as information provision, not advice. Concrete wording guidance:
- Acceptable: "Based on your profile, the following factors favor PKV: ..."
- Not acceptable: "We recommend you switch to PKV with provider X tariff Y"
- Always end with: "Consult a licensed independent insurance advisor (Versicherungsmakler) before making this decision"

---

## Sources

- PKV premium development and Altersrückstellungen: https://marcusknispel.com/beitragsentwicklung-pkv/, https://www.ottonova.de/v/private-krankenversicherung/beitragsentwicklung
- 2026 PKV threshold changes: https://feather-insurance.com/blog/private-health-insurance-threshold-yearly-changes
- GKV vs PKV family coverage: https://www.how-to-germany.com/health-insurance/private-health-insurance/families-children/, https://www.thegoodbroker.de/post/private-health-insurance-for-families-in-germany
- GKV return lock-in after 55: https://www.cp-finanz.de/krankenversicherung/zurueck-in-die-gkv-ab-55, https://www.clark.de/private-krankenversicherung/rueckkehr-gkv/
- Beamte and Beihilfe: https://beamtenberatung-plus.de/ratgeber/pkv-beamte/rueckkehr-gkv-beamte-wege-zurueck-in-die-gesetzliche-krankenversicherung/
- Anwartschaft for expats: https://www.waysafeinsurance.com/en/blog/health-insurance-anwartschaft-expats
- Common PKV switching mistakes: https://financeforexpats.de/news/why-you-should-not-switch-to-private-health-insurance-in-germany
- Versicherungspflichtgrenze 2026 (~€77,400): https://feather-insurance.com/blog/private-health-insurance-threshold-yearly-changes

---
*Pitfalls research for: v1.2 health insurance advisor — PKV vs GKV comparison*
*Researched: 2026-04-06*
