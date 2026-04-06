# Real Estate Investment Analysis - Financial Advisor Briefing
**Date:** {{DATE}}
**Status:** Decision Pending - Advisor Review Requested
**Property Type:** {{PROPERTY_TYPE}}

---

## EXECUTIVE SUMMARY

### What I'm Considering
{{INVESTMENT_DESCRIPTION}}

### Options Under Analysis

| Unit | Price | Upfront | Size | Yield | Parking | 10-Year Profit | ROE |
|------|-------|---------|------|-------|---------|----------------|-----|
{{UNITS_TABLE}}

### My Financial Profile
- **Gross income:** {{GROSS_INCOME}}/year
- **Marginal tax rate:** {{MARGINAL_TAX_RATE}}%
- **Liquid assets:** {{LIQUID_ASSETS}}
- **Monthly commitments:** {{MONTHLY_COMMITMENTS}}

### Why This Investment Works
1. **Years 1-4:** Cashflow {{CASHFLOW_Y1_4}} due to tax benefits
2. **Years 5-10:** Cashflow {{CASHFLOW_Y5_10}} - need to cover out-of-pocket
3. **Year 10:** Sell tax-free, profit of {{PROFIT_RANGE}}

### Main Concerns to Discuss

{{MAIN_CONCERNS}}

---

## PRIORITY QUESTIONS FOR ADVISOR

### 🔴 Urgent (Need Answer Before Deciding)

**1. Interest Rate - Can I Get Better?**
> Developer uses {{DEVELOPER_RATE}}%, but I researched and market offers {{MARKET_RATE}} for good profiles.
> With my income ({{GROSS_INCOME}}) and assets ({{LIQUID_ASSETS}}), what rate should I get?
> Potential savings: {{INTEREST_SAVINGS}} over 10 years.

**2. Are the Tax Calculations Correct?**
> I used marginal rate of {{MARGINAL_TAX_RATE}}% (42% + 5.5% Soli, no church tax).
> Is Sonder-AfA §7b of 5% for 4 years still valid for new construction?
> Are the calculated tax benefits ({{TAX_BENEFIT_RANGE}}/month in Years 1-4) realistic?

**3. Liquidity - Do I Have Enough Reserve?**
> After purchase, {{REMAINING_LIQUIDITY}} liquid remains.
> Years 5-10 require {{ANNUAL_OUTLAY_Y5_10}}/year contribution.
> How much should I keep as emergency reserve?

**4. Financing Structure**
> Is {{CURRENT_FINANCING}} ideal, or should I contribute more equity for better rate?
> Is {{FIXED_TERM}} years fixed adequate, or should I consider {{ALT_TERM}} years?

### 🟡 Important (To Optimize the Decision)

**5. Which Unit Do You Recommend?**
{{UNIT_OPTIONS}}

**6. Opportunity Cost**
> If I invest {{ALT_INVESTMENT}} in ETFs for 10 years at 7%, I'd have ~{{ETF_RETURN}}.
> The property projects {{PROJECTED_PROFIT}} profit + appreciation.
> Which makes more sense for my profile?

**7. Diversification**
> Does this investment represent excessive concentration in a single asset?
> Should I consider a smaller unit to diversify later?

**8. Early Sale Scenario**
> If I need to sell before 10 years, what's the tax impact?
> What's the penalty for breaking the financing?

### 🟢 For Future Planning

**9. Second Property**
> If this investment works, how soon could I consider a second?
> What would be the impact on my financing capacity?

**10. Refinancing at Year 10**
> If rates rise to {{STRESS_RATE}}%, what would be the impact?
> Should I plan to sell or refinance?

---

## 1. Investor Profile

### Personal Data
| Item | Value |
|------|-------|
| Tax residence country | {{COUNTRY}} |
| Tax status | {{TAX_STATUS}} |
| Tax class | {{TAX_CLASS}} |
| Church tax | {{CHURCH_TAX}} |
| Children | {{CHILDREN}} |

### Income
| Source | Annual Gross |
|--------|--------------|
{{INCOME_TABLE}}
| **Combined gross total** | **{{GROSS_INCOME}}** |

### Marginal Tax Rate Calculation
```
Income bracket: Above €68,480 → 42% base rate
Solidaritätszuschlag (solidarity surcharge): 42% × 5.5% = 2.31%
────────────────────────────────────────────────────────────────────
EFFECTIVE MARGINAL RATE: {{MARGINAL_TAX_RATE}}%
```

### Current Financial Position
| Asset | Value |
|-------|-------|
{{ASSETS_TABLE}}
| **Total liquid assets** | **{{LIQUID_ASSETS}}** |

### Monthly Commitments
| Item | Monthly |
|------|---------|
{{COMMITMENTS_TABLE}}
| **Total monthly contributions** | **{{MONTHLY_COMMITMENTS}}** |

### Investment Strategy
- **Goal:** {{INVESTMENT_GOAL}}
- **Horizon:** {{HORIZON}} years (Spekulationsfrist - capital gains tax exemption period)
- **Financing:** {{FINANCING_STRATEGY}}
- **Management:** {{MANAGEMENT_STRATEGY}}

---

## 2. Property Location: {{LOCATION}}

### Development: {{DEVELOPMENT_NAME}}
| Item | Details |
|------|---------|
{{DEVELOPMENT_TABLE}}

### Public Transport
| Item | Details |
|------|---------|
{{TRANSPORT_TABLE}}

### Parking Situation
{{PARKING_DESCRIPTION}}

### Why {{LOCATION}} (vs Other Locations Analyzed)

**Locations EXCLUDED from consideration:**

| Location | Exclusion Reason |
|----------|------------------|
{{EXCLUSIONS_TABLE}}

---

## 3. Shortlisted Properties

### General Comparison

| Metric | {{UNIT_HEADERS}} |
|--------|{{SEPARATORS}}|
{{GENERAL_COMPARISON_TABLE}}

---

## 4. Financing Assumptions (All Units)

| Parameter | Value | Source |
|-----------|-------|--------|
| Loan-to-value ratio | {{LTV}} | {{LTV_SOURCE}} |
| Interest rate (Zins) | {{INTEREST_RATE}}% | {{RATE_SOURCE}} |
| Repayment rate (Tilgung) | {{TILGUNG}}% | {{TILGUNG_SOURCE}} |
| Monthly payment formula | {{PAYMENT_FORMULA}} | Interest + Principal |
| Loan term | {{LOAN_TERM}} years fixed | Assumed |
| Construction period | {{CONSTRUCTION_PERIOD}} months | {{CONSTRUCTION_SOURCE}} |
| Average draw during construction | {{AVG_DRAW}}% | Conservative estimate |

### ⚠️ ALERT: Interest Rate May Not Be Optimized

**The {{INTEREST_RATE}}% rate used in developer calculations may be ABOVE current market.**

#### Current Market Rates ({{RESEARCH_DATE}})

| Fixed Period | Top Rate (Best Credit) | Typical Range |
|--------------|------------------------|---------------|
{{MARKET_RATES_TABLE}}

**Sources:** {{RATE_SOURCES}}

#### Comparison: Developer Rate vs Market

```
Developer rate:           {{INTEREST_RATE}}%
Current top rate:         {{TOP_RATE}}
Current typical range:    {{TYPICAL_RANGE}}
─────────────────────────────────────────────────────
DIFFERENCE:               {{RATE_DIFFERENCE}}
```

#### Financial Impact of Rate Difference

{{RATE_IMPACT_BY_UNIT}}

#### Why Investor May Get Better Rate

{{BETTER_RATE_REASONS}}

#### Rate Forecast for {{FORECAST_YEAR}}

{{RATE_FORECAST}}

#### Recommendation: Negotiate the Rate

**Institutions to quote:**
{{INSTITUTIONS_LIST}}

**Realistic target rate: {{TARGET_RATE}}**

#### Interest Rate Conclusion

{{INTEREST_RATE_CONCLUSION}}

---

## 5. Initial Capital Requirements

### Nebenkosten (Acquisition Costs) Breakdown
| Component | Rate | Notes |
|-----------|------|-------|
| Grunderwerbsteuer (transfer tax) | {{TRANSFER_TAX}}% | State transfer tax for {{STATE}} |
| Notar (notary) | {{NOTARY}}% | Legal fees |
| Grundbuch (land registry) | {{REGISTRY}}% | Registration fees |
| **Total Nebenkosten** | **{{TOTAL_NK}}%** | NOT recoverable, NOT financed, NOT deductible |

### Interest During Construction Phase
During {{CONSTRUCTION_PERIOD}} months of construction, investor pays interest on drawn loan but receives NO rent.
```
Formula: (Purchase price × {{AVG_DRAW}}% × {{INTEREST_RATE}}% interest) × {{CONSTRUCTION_YEARS}} years
```

### Total Initial by Unit

| Component | {{UNIT_HEADERS}} |
|-----------|{{SEPARATORS}}|
{{INITIAL_CAPITAL_TABLE}}

---

## 6. Monthly Cashflow Analysis

### Operating Cost Assumptions
| Item | Estimate | Notes |
|------|----------|-------|
| Verwaltung (management) | €{{VERWALTUNG_M2}}/m²/month | Professional WEG + SE management |
| Rücklage (reserve fund) | €{{RUCKLAGE_M2}}/m²/month | Building maintenance reserve |
| Vacancy provision | {{VACANCY}}% | Conservative: not included |
| Rent increase | {{RENT_INCREASE}}% | Conservative: not assumed |

{{CASHFLOW_BY_UNIT}}

---

## 7. Tax Benefit Methodology

### ⚠️ IMPORTANT: AfA is a Paper Deduction, NOT a Payment

**AfA (Absetzung für Abnutzung)** = Depreciation for tax purposes

This is a **non-cash deduction** that the German tax system allows because buildings theoretically lose value over time. **You do NOT pay AfA - it's a pure tax benefit.**

```
WHAT YOU PAY (Real Cash):             WHAT YOU DEDUCT (Paper Only):
────────────────────────────────      ────────────────────────────────────
✓ Mortgage interest                   ✓ Sonder-AfA (5%)  ← PAY NOTHING
✓ Principal repayment (Tilgung)       ✓ Regular AfA (2%) ← PAY NOTHING
✓ Verwaltung (management)
✓ Rücklage (reserve fund)

= CASH LEAVES YOUR ACCOUNT            = NO CASH LEAVES
                                        ONLY REDUCES TAXES
```

### How It Works in Practice (Example)

**REAL Monthly Expenses (cash leaving account):**
```
Mortgage payment:           €X,XXX/month  ← Real cash
Verwaltung:                 €XX/month     ← Real cash
Rücklage:                   €XX/month     ← Real cash
─────────────────────────────────────────
TOTAL REAL EXPENSES:        €X,XXX/month
```

**Deductions on Tax Return (annual):**
```
Interest paid:              €X,XXX/year   ← You actually paid this
Verwaltung:                 €XXX/year     ← You actually paid this
Sonder-AfA (5%):            €XX,XXX/year  ← PAPER ONLY - no cost
Regular AfA (2%):           €X,XXX/year   ← PAPER ONLY - no cost
─────────────────────────────────────────
TOTAL DEDUCTIONS:           €XX,XXX/year
```

**The AfA is FREE MONEY for tax purposes!**

### Why AfA is So Powerful

| Scenario | Tax Loss | Refund ({{MARGINAL_TAX_RATE}}%) |
|----------|----------|--------------------------------|
| **WITHOUT AfA** (interest + verwaltung only) | -€X,XXX/year | €XXX/year = €XX/month |
| **WITH AfA** (+ depreciation) | -€XX,XXX/year | €X,XXX/year = €XXX/month |
| **DIFFERENCE** | | **+€XXX/month EXTRA** |

**AfA depreciation generates €XXX/month additional refund without you spending anything!**

### Summary: What I Pay vs What I Deduct

| Item | Pay? | Deduct? | Note |
|------|------|---------|------|
| Mortgage interest | ✅ YES | ✅ YES | Real expense and deductible |
| Principal (Tilgung) | ✅ YES | ❌ NO | Paid but not deductible (it's equity) |
| Verwaltung | ✅ YES | ✅ YES | Real expense and deductible |
| Rücklage | ✅ YES | ❌ NO | Paid but not deductible (building reserve) |
| **Sonder-AfA (5%)** | **❌ NO** | **✅ YES** | Pure tax benefit |
| **Regular AfA (2%)** | **❌ NO** | **✅ YES** | Pure tax benefit |

---

### Sonder-AfA §7b EStG (Special Depreciation)
- **Eligibility:** New construction, application submitted before 2027
- **Rate:** 5% of building cost per year
- **Duration:** Maximum 4 years
- **Building portion:** ~92-94% of purchase price (excluding land)
- **Cumulative:** 20% depreciated in first 4 years

### Regular AfA (Standard Depreciation)
- **Rate:** 2% of building cost per year
- **Duration:** 50 years (for buildings completed after 2023)
- **Note:** Runs parallel to Sonder-AfA in years 1-4

### Combined Depreciation Years 1-4
```
Annual depreciation Years 1-4 = 5% (Sonder-AfA) + 2% (Regular AfA) = 7% of building
```

### Refund Timeline (Critical for Liquidity Planning)
```
Year 1 (Jan-Dec):     Pay expenses monthly out-of-pocket
Year 2 (March):       File tax return for Year 1
Year 2 (Sep-Dec):     Receive refund (9-18 months AFTER expenses paid)
```

**This creates a liquidity gap that must be financed from reserves.**

---

## 8. Exit Analysis (Year 10)

### Assumptions
| Parameter | Value | Notes |
|-----------|-------|-------|
| Property appreciation | {{APPRECIATION}}% per year | Conservative estimate |
| Sale costs | {{SALE_COSTS}}% of sale price | Broker, notary, fees |
| Capital gains tax | 0% | Exempt after 10 years (Spekulationsfrist) |

### Exit Calculations

| Metric | {{UNIT_HEADERS}} |
|--------|{{SEPARATORS}}|
{{EXIT_TABLE}}

### Cumulative Cash Position Over 10 Years

{{CASH_POSITION_BY_UNIT}}

---

## 9. Summary Comparison Table

| Metric | {{UNIT_HEADERS}} | Notes |
|--------|{{SEPARATORS}}|-------|
{{FULL_COMPARISON_TABLE}}

---

## 10. Risk Analysis

### Identified Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
{{RISKS_TABLE}}

### Stress Test: 0% Property Appreciation

If property values remain flat for 10 years:

| Metric | {{UNIT_HEADERS}} |
|--------|{{SEPARATORS}}|
{{STRESS_0_APPRECIATION_TABLE}}

**Conclusion:** {{STRESS_APPRECIATION_CONCLUSION}}

### Stress Test: Higher Interest Rate at Refinancing

If interest rates rise to {{STRESS_RATE}}% at Year 10 refinancing:
{{STRESS_INTEREST_ANALYSIS}}

---

## 11. Investor Suitability Assessment

### This Investment IS SUITABLE If:
{{SUITABLE_LIST}}

### This Investment May NOT Be Suitable If:
{{NOT_SUITABLE_LIST}}

---

## 12. Decision Framework for Advisor Discussion

{{DECISION_FRAMEWORK_BY_UNIT}}

### Parking Consideration
{{PARKING_CONSIDERATION}}

---

## 13. Questions for Financial Advisor

### Tax Questions
1. **Tax verification:** Are the Sonder-AfA §7b calculations correct for this income level ({{GROSS_INCOME}} gross, {{MARGINAL_TAX_RATE}}% rate)?
2. **Tax optimization:** Is there a more tax-efficient structure for this investment?

### Financing Questions
3. **Interest rate:** The {{INTEREST_RATE}}% rate proposed by developer is above current market ({{MARKET_RATE}} for best profiles). With my income and assets, what rate should I negotiate?
4. **Quote strategy:** Should I use a financing broker (Interhyp, Dr. Klein) or negotiate directly with banks?
5. **Financing structure:** Is {{LTV}} ideal, or should I contribute more equity for better rate?
6. **Fixed period:** Is {{LOAN_TERM}} years ideal, or should I consider {{ALT_TERM}} years given investment horizon?

### Liquidity Questions
7. **Liquidity planning:** Is current reserve ({{LIQUID_ASSETS}}) sufficient given negative cashflow needs in Years 5-10?
8. **Emergency buffer:** How much should I keep in liquid reserve after investment?

### Portfolio Questions
9. **Portfolio fit:** How does this real estate investment fit with my existing strategy?
10. **Opportunity cost:** What are opportunity costs vs keeping funds in ETFs with average 7% return?
11. **Diversification:** Does this investment represent excessive concentration in a single real estate asset?

### Risk Questions
12. **Risk tolerance:** Given stress test showing loss with 0% appreciation, is {{APPRECIATION}}% assumption reasonable for {{LOCATION}}?
13. **Time horizon:** Is there risk of needing capital before 10 years? What are tax consequences of early sale?
14. **Refinancing:** If rates rise to {{STRESS_RATE}}% at Year 10, what's the impact? Should I plan to sell or refinance?

### Interest Rate Question (PRIORITY)

**Context:** Developer uses {{INTEREST_RATE}}% in calculations, but market research ({{RESEARCH_DATE}}) shows:
{{MARKET_RATE_CONTEXT}}

**Question:** Given my profile (income {{GROSS_INCOME}}, assets {{LIQUID_ASSETS}}, no significant debts):
- What realistic rate should I achieve?
- Which banks/brokers do you recommend for best rate?
- Does the difference justify negotiation effort?
- Should I wait for rates to drop or lock in now?

---

## 14. Appendices

### A. Main Contacts
{{MAIN_CONTACTS}}

### B. Source Documents (Available in Project Folder)
{{SOURCE_DOCUMENTS}}

### C. Locations Analyzed and Excluded
| Location | Developer | Exclusion Reason |
|----------|-----------|------------------|
{{EXCLUDED_LOCATIONS_TABLE}}

### D. German Terms Glossary
| Term | English | Notes |
|------|---------|-------|
| Neubau | New construction | Newly built property, not existing |
| Nebenkosten | Acquisition costs | Transfer tax, notary, registration |
| Grunderwerbsteuer | Transfer tax | Similar to stamp duty |
| Sonder-AfA | Special depreciation | §7b EStG, 5%/year for 4 years |
| AfA | Regular depreciation | 2%/year for 50 years |
| Kaltmiete | Cold rent | Base rent excluding utilities |
| Verwaltung | Property management | WEG (building) + SE (unit) |
| Rücklage | Reserve fund | Building maintenance reserve |
| Tilgung | Principal repayment | Loan payment |
| Spekulationsfrist | Speculation period | 10 years; sale after is tax-free |
| Erbpacht | Ground lease | Leasehold, not freehold |
| Stellplatz | Parking space | Underground garage in this case |

---

## 15. Certification

This analysis was prepared based on:
- Price lists and documentation provided by developer
- Standard German tax calculations with {{MARGINAL_TAX_RATE}}% marginal rate
- Conservative assumptions ({{APPRECIATION}}% appreciation, 0% rent increase, 0% vacancy)
- Independent calculation methodology (not relying on developer marketing materials)

**All calculations should be verified by a qualified tax advisor before investment decision.**

---

**Document prepared:** {{DATE}}
**Status:** AWAITING FINANCIAL ADVISOR REVIEW
**Next step:** Schedule consultation with advisor to review analysis and finalize decision

---

## 16. Executive Summary

```
════════════════════════════════════════════════════════════════════════════════
                    FINAL SHORTLIST - {{DEVELOPMENT_NAME}}
════════════════════════════════════════════════════════════════════════════════

{{ASCII_SUMMARY_TABLE}}

════════════════════════════════════════════════════════════════════════════════
INVESTOR PROFILE
════════════════════════════════════════════════════════════════════════════════
Gross income:          {{GROSS_INCOME}}/year
Marginal tax rate:     {{MARGINAL_TAX_RATE}}%
Liquid assets:         {{LIQUID_ASSETS}}
Strategy:              {{STRATEGY_SUMMARY}}

════════════════════════════════════════════════════════════════════════════════
                    STATUS: DECISION PENDING
════════════════════════════════════════════════════════════════════════════════
```

---

# AGENT INSTRUCTIONS

## How to Use This Template

This template should be filled with specific data from each investment analysis. Placeholders are marked with `{{VARIABLE_NAME}}`.

## Language

- **Default:** English (this template)
- **If user requests different language:** Translate the entire document while maintaining structure
- **Common request:** Brazilian Portuguese for this investor's financial advisor

## Required Sections

1. **Executive Summary** - Always at top, with options table and profile
2. **Priority Questions** - Organized by urgency (🔴/🟡/🟢)
3. **Investor Profile** - Data from AGENTS.md
4. **Location** - Development details and exclusions
5. **Unit Comparison** - Detailed tables
6. **Financing Assumptions** - With interest rate alert
7. **Cashflow** - By unit, Years 1-4 and 5-10
8. **AfA Explanation** - CRITICAL: Explain it's a paper deduction
9. **Exit Analysis** - Year 10 calculations
10. **Comparison Table** - All metrics
11. **Risk Analysis** - With stress tests
12. **Decision Framework** - "Choose X if..."

## Filling Rules

1. **Numbers:** Use appropriate format for target language
2. **Percentages:** Use appropriate decimal separator
3. **Tables:** Always include all units under analysis
4. **AfA:** ALWAYS explain it's not a payment
5. **Interest rate:** ALWAYS compare with current market
6. **Stress tests:** ALWAYS include 0% appreciation scenario

## Investor Variables (from AGENTS.md)

Always use updated data from investor profile in AGENTS.md:
- Income, assets, monthly commitments
- Marginal tax rate (currently 44.31%)
- Investment strategy

## Interest Rate Sources

Always research current rates before generating document:
- Dr. Klein (drklein.de)
- Baufi24 (baufi24.de)
- Interhyp (interhyp.de)
- Finanztip (finanztip.de)
