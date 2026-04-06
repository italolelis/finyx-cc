# Feature Landscape

**Domain:** Personal finance advisor — tax optimization, investment advisory, portfolio management, broker comparison, pension planning (Germany + Brazil)
**Researched:** 2026-04-06
**Confidence:** MEDIUM-HIGH (German tax ecosystem well-documented; Brazilian ecosystem sourced from Portuguese-language financial press + official sources)

---

## Table Stakes

Features users expect. Missing = product feels incomplete or untrustworthy.

### Tax Advisory

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Tax class explanation (Germany) | Every German employee faces Steuerklassen; confusion is universal | Low | Static reference + profile-aware mapping |
| Sparerpauschbetrag tracking | €1,000/€2,000 exemption is universally applicable; users want to know how much they've used | Low | Requires user to input holdings/dividends |
| Vorabpauschale calculation | Applies annually to all accumulating ETFs; most users don't know how to calculate it | Medium | Requires Basiszins (published by Bundesbank) + fund type |
| Teilfreistellung awareness | Affects how ETF income is taxed; wrong calculation = wrong DARF/Anlage KAP | Low | Reference table by fund type |
| Abgeltungssteuer summary | 25% + Soli + KiSt is baseline German capital gains tax; users must understand it | Low | Explain with user's marginal rate context |
| Brazilian IR filing guidance | Filing season complexity; users need step-by-step for investment types | Medium | Stocks, FIIs, CDB, LCI/LCA, previdência all have different rules |
| DARF calculation and reminder | Monthly obligation for stock/FII gains; common compliance failure | Medium | Calculate monthly taxable gains, remind before last business day |
| Come-cotas explanation | Biannual tax on investment funds confuses users; often discovered at redemption | Low | Explain mechanism + timing |
| FII dividend tax exemption rules | Exempt but must be declared; common misconception they're invisible to Receita | Low | Explain declaration requirement |
| Legal disclaimer on all advice | Advisory-only tool; must not imply professional tax advice | Low | Mandatory for all outputs |

### Investment Advisory

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Portfolio allocation overview | Users want to know where their money is (geography, sector, asset class) | Medium | Requires user to input holdings |
| Risk profile assessment | Underpins all investment recommendations; robo-advisors all do this | Low | Questionnaire + mapping to risk tolerance |
| ETF recommendation by goal | ETF investing is dominant in Germany (Sparplan culture); users expect suggestions | Medium | Must be profile-aware, not generic |
| Asset class explanation | Stocks vs ETFs vs futures vs FIIs vs CDBs — users often conflate them | Low | Reference content per asset type |
| Rebalancing suggestion | Drift alerts (e.g., equity % too high) are expected by any portfolio tool | Medium | Requires target allocation baseline |
| Market data for queried assets | Users asking about a stock/ETF expect current price, basic metrics | Medium | Yahoo Finance / web search fallback |

### Broker Comparison

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Fee comparison (Germany) | Trade Republic vs Scalable vs ING vs comdirect — every expat/investor asks this | Low | Structured reference data, updated per year |
| Fee comparison (Brazil) | NuInvest zero-fee vs XP vs BTG is a common decision point | Low | Structured reference data |
| Profile-based recommendation | Not "here are all brokers" but "given your situation, use X" | Medium | Depends on user profile (country, trade frequency, asset types) |
| Tax reporting quality note | German brokers differ significantly on Anlage KAP auto-generation; critical for expats using foreign brokers | Low | Key factor: IBKR/DEGIRO users must calculate manually |
| Cross-border account guidance | IBKR is common for DE+BR users; Schwab for US; needs explanation | Medium | Regulatory, tax, and reporting implications |

### Pension Planning

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Riester vs Rürup vs bAV comparison | Every German employee/freelancer faces this choice; no intuitive right answer | Medium | Must factor employment status, tax bracket, family subsidies |
| Riester Zulagen calculator | State subsidies (Grundzulage + Kinderzulage) are a core Riester selling point | Medium | Requires family data from user profile |
| Rürup tax deduction estimate | Primary benefit is Sonderausgabenabzug; users want to see the savings | Medium | Requires income and marginal tax rate |
| bAV basics explanation | Entgeltumwandlung reduces gross income; misunderstood by most employees | Low | Explanation + rule of thumb |
| PGBL vs VGBL decision guide | Most-asked Brazilian pension question; wrong choice costs significant tax | Medium | Depends on IR regime and 12% income threshold |
| PGBL 12% contribution limit guidance | PGBL deduction cap is 12% of annual taxable income; common source of errors | Low | Calculate from user income |
| Progressive vs regressive IR regime for previdência | Regime choice is irrevocable per contract; high stakes | Low | Explain with time horizon examples |
| Pension gap estimate | "How much do I need vs what I'll have" is the core planning question | High | Requires projection model with inflation and contribution assumptions |

---

## Differentiators

Features that set the product apart. Not expected, but valued.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Integrated cross-country profile | Advise someone who lives in Germany but invests in Brazil simultaneously — no other tool does this | High | Core differentiator of Finyx; requires shared memory system |
| German+Brazilian tax interaction | E.g., Brazilian PGBL deduction + German Anlage AUS for foreign income — truly rare advisory capability | High | Needs both reference systems loaded simultaneously |
| Tax-loss harvesting guidance (Germany) | Identify unrealized losses in portfolio to offset gains before year-end; not common in DE tools | High | Requires holdings data + current prices |
| Tax-loss harvesting guidance (Brazil) | Isenção de R$20k/month for stocks — help users plan sells to stay under threshold | Medium | High value; commonly missed |
| Vorabpauschale pre-calculation for year | Run before Jan 2 to know cash needed for auto-debit; prevents account shortfall | Medium | Differentiating — most users are caught off guard |
| Sparerpauschbetrag optimization across brokers | If user has multiple brokers, help distribute Freistellungsauftrag optimally | Medium | Common pain point for multi-broker investors |
| PFOF ban impact analysis (June 2026) | EU PFOF ban changes neo-broker economics; advisory content on what changes for users | Low | Timely, relevant, rare in tools |
| Real estate + investment integration | Advise on Eigennutzung vs Vermietung tax implications alongside investment portfolio | High | Unique because of IMMO lineage |
| Broker migration guide | Help user understand tax and reporting consequences of switching brokers | Medium | Practical, avoids painful surprises |
| Brazil Bolsa de Valores isenção planning | Month-by-month sell planning to stay under R$20k exemption ceiling | Medium | High value for active Brazilian investors |
| DARF reminder + calculation workflow | End-to-end monthly flow: calculate gain → confirm DARF amount → remind before deadline | Medium | Removes compliance anxiety |
| Country-specific pension projection | Show retirement income simulation with DE statutory pension + private + BR INSS | High | Requires actuarial assumptions, but powerful for expats |

---

## Anti-Features

Features to deliberately NOT build.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Tax return filing automation | Legal liability; errors cause penalties; out of scope in PROJECT.md | Guide users step-by-step; link to ELSTER / Receita Federal tools |
| Automated trade execution | Securities law in both countries; advisory-only is the explicit constraint | Provide recommendations; user executes at broker |
| Real-time portfolio sync via broker APIs | High complexity, OAuth maintenance, API instability, out of scope v1 | User inputs holdings manually or pastes CSV |
| Insurance comparison engine | Data sourcing complexity (acknowledged in PROJECT.md as v2) | Mention insurance as a consideration; don't build comparison |
| Budget tracking / expense categorization | Check24/Mint-style feature; different product; would dilute focus | Refer users to dedicated tools (YNAB, etc.) |
| Robo-advisor / automated rebalancing | Requires broker integration + execution + regulatory licensing | Provide rebalancing analysis + instructions; user acts |
| Countries beyond Germany and Brazil | Architecture bloat; v1 scope is clear in PROJECT.md | Keep reference system modular so community can add countries |
| Financial product selling / affiliate links | Destroys trust; conflicts with advisory-only positioning | Pure advisory, never recommend products for commercial reasons |
| Chat history / conversation persistence between sessions | Claude Code handles context naturally; building custom persistence is over-engineering | Use Claude's native context; GSD memory system for profile |
| Push notifications / alerting system | Requires infrastructure (backend, scheduler); CLI tool, not a service | User runs commands; no daemon/scheduler in v1 |
| Social features / community portfolios | Off-brand for a private financial advisor | Stay single-user, private-by-design |

---

## Feature Dependencies

```
User Financial Profile
  → German Tax Advisor (needs: income, Steuerklasse, employment type, Kirchensteuer)
  → Brazilian Tax Advisor (needs: residency status, income, investment types held)
  → Investment Advisor (needs: risk profile, goals, current holdings)
  → Broker Comparison (needs: country, trading frequency, asset types, tax complexity)
  → Pension Planning (needs: employment status, income, age, family status, country)

German Tax-Aware Investing
  → Requires: Sparerpauschbetrag allocation + current holdings + broker setup
  → Requires: Vorabpauschale reference data (Basiszins published annually by Bundesbank)

Brazilian Tax-Aware Investing
  → Requires: Monthly trade history (for isenção calculation)
  → Requires: Asset type breakdown (stocks vs FIIs vs funds have different rules)

Pension Planning (Germany)
  → Riester: needs family status + income + employment type (not for Selbständige)
  → Rürup: needs income + tax bracket (primary benefit is Sonderausgabenabzug)
  → bAV: needs employer + employment status

Pension Planning (Brazil)
  → PGBL/VGBL: needs IR regime (progressive vs regressive) + investment horizon + income
  → INSS: needs contribution history (user-provided)

Portfolio Tracker
  → Rebalancing: requires target allocation (set in profile or per-session)
  → Tax-loss harvesting: requires current prices + cost basis

Broker Comparison
  → Germany: profile must know if user uses foreign broker (IBKR, DEGIRO) — different tax reporting burden
  → Brazil: profile must know if user needs BDRs, mini-contracts, or only basic assets
```

---

## MVP Recommendation

Build in this order to deliver integrated value incrementally:

1. **User Financial Profile** — shared memory system; gates all other features
2. **German Tax Advisor** — highest-demand, well-documented rules, existing IMMO tax knowledge to build on
3. **Brazilian Tax Advisor** — second country; validates multi-country architecture
4. **Investment Advisor + Portfolio Tracker** — core value for investors; builds on profile
5. **German Tax-Aware Investing** — differentiator; high value once portfolio data exists
6. **Brazilian Tax-Aware Investing** — isenção planning + DARF workflow; high practical value
7. **Broker Comparison** — relatively static reference data; build as reference command
8. **Pension Planning** — highest complexity; requires all profile data to be meaningful

**Defer:**
- Tax-loss harvesting guidance: requires accurate cost basis + current prices — build after portfolio tracker is solid
- Pension gap projection: actuarial complexity; build after basic pension planning is validated
- DARF automation workflow: build after Brazilian tax advisor is working

---

## Sources

- [Best Broker for Expats in Germany 2026](https://perfinex.de/best-broker-expats-germany-2026-guide/)
- [Investing Taxes Germany: Vorabpauschale, CGT & ETF Guide 2026](https://quantroutine.com/learn/investing-taxes-germany/)
- [German Tax Tracker — Vorabpauschale Calculator](https://www.germantaxtracker.de/)
- [Capital Gains Tax Germany 2026: Abgeltungssteuer Guide](https://www.allinvestview.com/articles/capital-gains-tax-germany/)
- [Germany's Vorabpauschale 2026: Basiszins, ETFs & tax](https://hexn.io/local-updates/vorabpauschale-2026-the-cost-of-a-passive-portfolio-in-germany-rp9h14bpv3epm9ch19mspw3a)
- [PGBL and VGBL: differences explained](https://www.dpc.com.br/pgbl-and-vgbl-understanding-the-differences-between-these-private-pension-plans/?lang=en)
- [Brazilian IR 2026: como declarar investimentos](https://www.idinheiro.com.br/investimentos/como-declarar-investimentos-no-imposto-de-renda/)
- [Como emitir e pagar DARF 2026](https://blog.nubank.com.br/como-emitir-darf-investimentos/)
- [XP Investimentos vs BTG Pactual 2026](https://arevista.com.br/mercados/ibovespa/xp-investimentos-ou-btg-pactual-qual-e-a-melhor-corretora-para-investir-em-2026/)
- [NuInvest vs BTG Pactual comparison](https://rankia.com.br/nuinvest-vs-btg-pactual-corretoras/)
- [Scalable Capital vs Trade Republic 2026](https://northern.finance/en/review/scalable-capital-vs-trade-republic/)
- [Riester vs Rürup: differences and suitability](https://xpert.digital/en/pension-differences/)
- [Rentenrechner Deutschland 2026](https://www.securefuturecalculator.com/retirement-calculator-germany)
- [AI Tools for Tax-Loss Harvesting](https://www.mezzi.com/blog/ai-tools-for-tax-loss-harvesting-in-portfolios)
- [PFOF ban June 2026: Commission-Free Brokers](https://www.bankeronwheels.com/pfof-and-quote-driven-reviews/)
- [Fintech UX Anti-Patterns 2026](https://theuxda.com/blog/top-20-financial-ux-dos-and-donts-to-boost-customer-experience)
