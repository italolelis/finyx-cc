# Domain Pitfalls

**Domain:** AI financial advisor — tax optimization, investment advice, pension planning (Germany + Brazil)
**Researched:** 2026-04-06

---

## Critical Pitfalls

### Pitfall 1: Stale Tax Rules Presented as Current

**What goes wrong:** Reference docs contain last-year's thresholds, rates, or rules. The AI uses them confidently with no uncertainty signal. User relies on advice, files incorrectly, or misses a deduction.

**Why it happens:** Tax rules change annually (Sparerpauschbetrag limits, Vorabpauschale base rate, Brazilian IR tables). Without an active update process, reference docs drift. The model cannot self-detect staleness — it has no calendar awareness in prompts.

**Consequences:** Wrong deduction claims, missed allowances, penalties, incorrect DARF amounts, stale Vorabpauschale base rate (e.g., 2025 rate = 2.53% — this changes every year via BMF circular). Brazil's Law 15,270/2025 (effective 2026-01-01) introduced dividend taxation and new exemption thresholds — an advisor unaware of this gives wrong guidance to every Brazilian user.

**Detection:**
- Reference docs lack a `last_updated` or `tax_year` frontmatter field
- No CI check comparing reference doc version to current tax year
- Advice contradicts known recent changes

**Prevention:**
- Every reference doc must include `tax_year:` and `valid_until:` metadata
- Add a stale-data warning gate in prompts: if `tax_year` != current year, surface explicit disclaimer before advice
- Create a `TAX_YEAR_CHECKLIST.md` that lists every value that changes annually (Germany: Grundfreibetrag, Sparerpauschbetrag, Vorabpauschale base rate; Brazil: IR brackets, FII exemption rules, come-cotas rates)
- Flag this in every phase that creates or uses reference docs

**Phase:** Affects Phase covering German Tax Advisor, Brazilian Tax Advisor, and every subsequent phase that reads reference docs. Address in the foundation phase.

---

### Pitfall 2: Wrong Jurisdiction Applied to Cross-Border Users

**What goes wrong:** A Brazilian expat living in Germany gets German-only advice, or vice versa. The advisor ignores that Brazil taxes residents on worldwide income and Germany has its own rules for foreign-sourced income.

**Why it happens:** Jurisdiction is inferred from a single field in the user profile ("country: Germany"). But the financial situation of a Brazilian resident in Germany involves both jurisdictions simultaneously — Brazilian stocks held by a German resident still require DARF for capital gains above the monthly exemption.

**Consequences:** Missed Brazilian DARF obligations, incorrect double-taxation treaty application, wrong Teilfreistellung assumptions for Brazilian ETFs, or treating Brazilian FII dividends as German Kapitalertragsteuer-exempt.

**Detection:**
- User profile has only one country field
- Advisor prompts branch on a single country without asking about tax residency history, dual obligations, or expat status

**Prevention:**
- User profile must capture: primary tax residency, secondary obligations (e.g., Brazilian worldwide income), expat status, years in current country
- When country fields suggest cross-border situation, always surface a "this may involve both jurisdictions" disclaimer before advice
- Reference docs for Germany and Brazil must include sections on cross-border treatment, not just domestic-only rules

**Phase:** User Financial Profile build phase. Must be addressed before any tax advisor phase.

---

### Pitfall 3: Confidence Signaling Without Certainty

**What goes wrong:** The AI presents advice with false confidence — no hedge, no "verify with a tax professional" — leading users to treat calculated estimates as filings-ready numbers.

**Why it happens:** LLMs default to confident prose. Without explicit prompt engineering, they don't surface uncertainty even when calculating complex values like Vorabpauschale (which depends on the published base rate and Teilfreistellungsquote) or Brazilian come-cotas (which depends on fund type and NAV).

**Consequences:** Users file self-assessment (Anlage KAP) using AI-calculated numbers that are wrong. Brazilian users submit incorrect DARF amounts. Legal exposure for the project regardless of disclaimers.

**Detection:**
- Agent prompts don't contain explicit uncertainty-signaling instructions
- Output never says "estimate" or "verify with [official source]"
- No disclaimer surface in final outputs

**Prevention:**
- Every advisor agent must include in its system prompt: explicit instructions to flag estimates vs. definitive values, and to always recommend professional verification for filing-ready numbers
- All generated reports must include a disclaimer block (not just in README, but in the output itself)
- Distinguish "orientation advice" (how does Vorabpauschale work) from "calculation" (your Vorabpauschale this year is X) — calculations must always be flagged as estimates

**Phase:** All advisor agent phases. Establish the pattern in the first agent built and enforce it as a template constraint.

---

### Pitfall 4: Freistellungsauftrag / Sparerpauschbetrag Miscalculation Across Brokers

**What goes wrong:** User holds accounts at multiple brokers (Trade Republic + ING + Scalable). The advisor calculates each account's tax exposure in isolation, not summing the Freistellungsauftrag allocation across brokers. Advice to "not set up a Freistellungsauftrag" at broker X misses that the user's remaining allowance is zero because it's already allocated at broker Y.

**Why it happens:** The advisor only sees data the user provides. If the profile captures only one broker, the cross-broker picture is invisible.

**Consequences:** User over-applies Sparerpauschbetrag (a tax offense per German law); or under-uses it (leaves money on the table).

**Detection:**
- Profile only captures a single broker
- Tax calculation doesn't ask "do you have other accounts?"
- No explicit multi-broker allocation prompt in the German tax advisor flow

**Prevention:**
- User profile must capture all active broker accounts
- German tax advisor workflow must include a mandatory multi-broker check step before calculating Sparerpauschbetrag allocation advice
- Include explicit warning: total Freistellungsauftrag across all brokers cannot exceed €1,000 (single) / €2,000 (married)

**Phase:** German Tax Advisor phase. Must be a required step in the advisory flow, not optional.

---

## Moderate Pitfalls

### Pitfall 5: Riester/Rürup Advice Without Full Tax Bracket Context

**What goes wrong:** The advisor recommends a Riester or Rürup product based on the stated contribution benefit without accounting for the deferred taxation at retirement. The effective benefit depends on the user's marginal rate now vs. expected rate in retirement.

**Why it happens:** Rürup contributions are 100% deductible in 2025, which looks attractive. But both Riester and Rürup require paying income tax on the full payout in retirement. The "Rentenfaktor" cost typically outweighs tax benefits unless the user's retirement income stays low.

**Prevention:**
- Pension advisor must calculate net benefit (contribution tax savings minus expected retirement tax cost) before making a recommendation
- Always surface the Rentenfaktor/cost comparison, not just the contribution deductibility headline
- Distinguish between users with low retirement income expectations (Riester/Rürup may be worth it) vs. high earners who will pay full tax at retirement

**Phase:** German Pension Planning phase.

---

### Pitfall 6: Brazilian FII Exemption Conditions Ignored

**What goes wrong:** The advisor tells users that Brazilian FII (Real Estate Investment Trust) dividends are tax-exempt, without checking the eligibility conditions. Exemption requires: at least 100 quota holders, and the individual must not hold 10%+ of total quotas OR generate 10%+ of fund income.

**Why it happens:** The headline rule ("FII dividends are exempt") is well-known. The conditions are less prominent.

**Consequences:** User treats taxable FII income as exempt; misses IRRF obligation; incorrect IR declaration.

**Prevention:**
- Brazilian investment reference docs must include the full exemption eligibility check, not just the headline rate
- FII analysis workflows must include a checklist: quota holder count (check fund report), user's ownership percentage
- Flag the 2025 rate reduction (20% → 17.5% for non-exempt cases) in reference docs

**Phase:** Brazilian Tax Advisor and Investment Advisor phases.

---

### Pitfall 7: Come-Cotas Liquidity Surprise for Accumulating Fund Users

**What goes wrong:** Users holding accumulating fixed-income or multimarket funds get told the tax is "handled automatically" without understanding they owe tax in May and November even if they have not sold anything and received no cash distributions.

**Why it happens:** Come-cotas is a prepayment mechanism — it's "fictitious income" tax. Users with accumulating funds need to ensure liquidity to pay without being forced to sell units.

**Prevention:**
- Brazilian tax advisor must surface come-cotas timing and liquidity requirement for all users holding qualifying funds
- Reference docs must clearly distinguish: funds subject to come-cotas (fixed-income, multimarket) vs. exempt (equity funds, FII, FIAGRO)

**Phase:** Brazilian Tax Advisor phase.

---

### Pitfall 8: Vorabpauschale Base Rate Hardcoded or Missing

**What goes wrong:** The Vorabpauschale calculation uses the wrong Basiszins. The base rate is published annually by the German Ministry of Finance (BMF). Using the wrong rate produces an incorrect tax estimate.

**Why it happens:** The rate is easy to hardcode during development. It changes every year and is not widely known outside Germany.

**Prevention:**
- Do NOT hardcode the Basiszins in reference docs without a `valid_for_tax_year` marker
- Add to the annual tax checklist: "Update Basiszins from BMF circular"
- Include source URL in reference docs: https://www.bundesfinanzministerium.de
- Prompt the user to verify the current year's base rate before accepting any Vorabpauschale estimate

**Phase:** German Tax Advisor / German Tax-Aware Investing phase.

---

### Pitfall 9: Prompt Injection via User Financial Data

**What goes wrong:** User profile fields (asset descriptions, notes) contain malicious instructions that hijack agent behavior. Claude Code has documented prompt injection vulnerabilities via subcommand chains.

**Why it happens:** Financial data is free-form text. A field like "investment notes" or "property description" is user-controlled input that gets embedded into agent prompts.

**Consequences:** Data exfiltration of user financial profile, agent executing unintended commands, incorrect advice generated from manipulated context.

**Detection:**
- Profile fields are passed raw into prompt templates without sanitization
- Agent prompts don't isolate user-supplied data from instructions

**Prevention:**
- Wrap all user-supplied profile fields in clear delimiters in prompts (e.g., `<user_data>` blocks with explicit instruction that this content is data, not instructions)
- Do not allow free-form text in high-trust fields (income, tax class) — use structured enums where possible
- Follow least-privilege principle: never grant `--dangerously-skip-permissions` in financial advisor commands

**Phase:** Shared Memory System / User Financial Profile phase. Must be addressed before any agent reads from the profile.

---

## Minor Pitfalls

### Pitfall 10: "Advisory Only" Positioning Eroded by Output Format

**What goes wrong:** Despite the tool being advisory-only, outputs formatted as "Action Plan" or "Filing Instructions" read as directives. Users follow them without further verification.

**Prevention:**
- Output templates must use language like "consider", "estimate", "this may apply to you" — not "you owe", "file by", "claim"
- Add a standard disclaimer block to every generated report that cannot be removed

**Phase:** All phases. Establish in the first agent, enforce as template standard.

---

### Pitfall 11: Brazil Tax Reform Lag (Law 15,270/2025)

**What goes wrong:** Advisor uses pre-2026 Brazilian tax rules — misses the new dividend withholding tax (10% on dividends >BRL 50k/month), the new individual income tax exemption (up to BRL 5,000/month), and the Interest on Net Equity rate increase (15% → 20%).

**Why it happens:** These rules took effect 2026-01-01 and are recent enough to be missing from any reference docs written before late 2025.

**Prevention:**
- Brazilian reference docs must be written to reflect Law 15,270/2025 from the start
- Include version metadata: `effective_from: 2026-01-01`
- Sources: KPMG flash alert (Law 15,270/2025), EY tax alert

**Phase:** Brazilian Tax Advisor phase. Reference docs must be current before the phase ships.

---

### Pitfall 12: Pension Advice Without Portability Check

**What goes wrong:** bAV (company pension) is recommended without flagging that portability is limited. If the user changes employers, transferring bAV can be complex or impossible depending on the product.

**Prevention:**
- German pension advisor must surface portability risk for bAV before recommending it
- Flag that bAV is primarily valuable for long-tenure employees

**Phase:** German Pension Planning phase.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|----------------|------------|
| User Financial Profile | Single-country assumption; missing multi-broker fields | Capture: tax residency, all brokers, expat status |
| German Tax Advisor | Stale Sparerpauschbetrag/Vorabpauschale rates; multi-broker Freistellungsauftrag split | Tax-year metadata; mandatory multi-broker check |
| Brazilian Tax Advisor | Law 15,270/2025 lag; FII exemption conditions; come-cotas liquidity | Write reference docs to 2026 rules from the start |
| Investment Advisor | Cross-jurisdiction Teilfreistellung; hallucinated market data | Source live data; flag estimates |
| Pension Planning (DE) | Riester/Rürup net-benefit oversimplification; bAV portability | Calculate net benefit; surface portability risk |
| Pension Planning (BR) | INSS rules for expats and contribution gaps | Research INSS expat treatment before building |
| Shared Memory System | Prompt injection via user-supplied fields | Delimit user data in prompt templates |
| All agents | Confidence without certainty; "advisory only" eroded by output | Enforce disclaimer template; use hedged language |

---

## Sources

- [FPA: Compliance Risks of Using Generative AI in Financial Planning](https://www.financialplanningassociation.org/learning/publications/journal/MAY25-compliance-risks-using-generative-ai-financial-planning-practice-OPEN)
- [Kitces: Inadvertent Tax Advice Liability](https://www.kitces.com/blog/tax-advice-liability-risk-advisor-tax-planning-value-add-value-strategy-financial-planning-clients/)
- [CNBC: I asked ChatGPT for tax help — classic trap](https://www.cnbc.com/2026/03/31/ai-tax-help-pitfalls.html)
- [Expat Wealth at Work: AI Financial Advice Risks](https://expatwealthatwork.com/blog/2026/03/12/ai-financial-advice-risks-the-costly-mistakes-nobody-warns-you-about/)
- [Upvest: Vorabpauschale guide](https://docs.upvest.co/products/tol/guides/taxes/tax_collection_vorabpauschale)
- [AllInvestView: Abgeltungssteuer guide 2026](https://www.allinvestview.com/articles/capital-gains-tax-germany/)
- [KPMG: Brazil Law 15,270/2025 — Individual Income Tax reform](https://kpmg.com/xx/en/our-insights/gms-flash-alert/flash-alert-2025-251.html)
- [EY: Brazil modifies investment fund taxation](https://www.ey.com/en_gl/technical/tax-alerts/brazil-modifies-how-investment-funds-will-be-taxed)
- [Gringo Investor: Brazilian FIIs guide](https://www.gringoinvestor.com/brazilian-reits/)
- [BaFin: Riester pensions](https://www.bafin.de/EN/Verbraucher/Altersvorsorge/Riester/riester_node_en.html)
- [SC Media: Claude Code prompt injection vulnerability](https://www.scworld.com/brief/claude-code-vulnerable-to-prompt-injection-due-to-subcommand-limit)
- [Tax Notes: Automated Tax Planning — Who's Liable When AI Gets It Wrong?](https://www.taxnotes.com/featured-analysis/automated-tax-planning-whos-liable-when-ai-gets-it-wrong/2023/09/22/7h97l)
