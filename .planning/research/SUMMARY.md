# Project Research Summary

**Project:** Finyx — Personal Finance Advisor CLI
**Domain:** Multi-country AI financial advisor (Germany + Brazil) as Claude Code slash-command plugin
**Researched:** 2026-04-06
**Confidence:** HIGH

## Executive Summary

Finyx extends the proven IMMO meta-prompting architecture — Markdown prompts, sub-agent `.md` files, JSON profile, versioned reference docs — into financial advisory domains. There is no new runtime, no new framework, and no npm dependencies to add. The architecture question is settled: everything that changes (tax rates, pension limits, broker fees) lives in reference docs; all logic is prompt-driven; all state flows through `.finyx/profile.json`. The pattern is validated by the existing codebase.

The recommended build order is profile-first, then Germany end-to-end, then Brazil as a parallel path, then investment advisory, then pension planning. This order reflects hard dependencies: every specialist command is gated on the user profile, and isolating one country reduces country-specific debugging complexity before adding the second. The cross-border expat case — a Brazilian living in Germany with obligations to both tax authorities — is the defining differentiator and must be addressed in the profile design, not retrofitted later.

The critical risks are not technical. They are: stale reference docs presenting incorrect tax rules as current fact; false confidence in AI-generated calculations (particularly Vorabpauschale, DARF amounts, and pension projections); and cross-border jurisdiction errors when a single "country" field is the only context. All three can be prevented through prompt engineering conventions established in the first agent built and enforced as templates thereafter. Legal exposure is real — disclaimers must be embedded in output, not only in documentation.

## Key Findings

### Recommended Stack

No new stack. The existing Node.js installer + Markdown prompt system handles everything. External data access uses `WebSearch`/`WebFetch` and `Bash(curl)` — no npm dependencies added.

For live market data, Finnhub (official Node.js SDK, 60 req/min free tier) is the primary source for US and European equities. brapi.dev covers B3 stocks and FIIs, including FII-specific metrics (P/VP, dividend yield) that Finnhub misses. Neither requires bundled credentials — users set environment variables. The Bundesbank SDMX API supplies the annual Basiszins needed for Vorabpauschale calculations, free and unauthenticated.

Tax rules and pension parameters are static reference Markdown files, updated by maintainers annually. No tax rule API exists for Germany or Brazil — this is by design and matches the existing IMMO pattern for AfA and Grunderwerbsteuer.

**Core data sources:**
- **Finnhub**: US/EU equities + ETFs — official SDK, free tier, 60 req/min
- **brapi.dev**: B3 stocks, FIIs, BDRs — free unauthenticated, FII-native
- **Bundesbank SDMX API**: Basiszins — free, no auth, programmatic
- **OpenFIGI**: ISIN-to-ticker mapping — free, Bloomberg-backed (Phase 2+)
- **Reference docs**: All tax rates, pension limits, broker fees — versioned Markdown, updated annually

**Explicitly avoided:** Yahoo Finance (ToS violation), Alpha Vantage (unreliable free tier), any server-side middleware.

### Expected Features

**Must have (table stakes):**
- Sparerpauschbetrag tracking — universally applicable to German investors
- Vorabpauschale calculation — applies annually to all accumulating ETFs; users don't know how to calculate it
- Abgeltungssteuer + Teilfreistellung explanation — baseline German capital gains context
- Brazilian DARF calculation and reminder — monthly compliance failure is common
- FII dividend exemption rules (with full conditions) — widely misunderstood
- Come-cotas explanation with liquidity timing — surprise tax event for accumulating fund holders
- Risk profile assessment — gates all investment recommendations
- ETF recommendation by goal — dominant investment pattern in Germany (Sparplan culture)
- Riester vs Rürup vs bAV comparison — every German employee/freelancer faces this
- PGBL vs VGBL decision guide — wrong choice is costly and common in Brazil
- Legal disclaimer on all outputs — non-negotiable

**Should have (differentiators):**
- Integrated cross-country profile — no other tool advises a DE-resident Brazilian investor simultaneously
- Sparerpauschbetrag optimization across multiple brokers — common multi-broker pain point
- Brazilian isenção planning (R$20k/month stock exemption) — high-value for active B3 investors
- Vorabpauschale pre-calculation before year-end — prevents account shortfall in January
- PFOF ban impact analysis (June 2026) — timely, rare in current tools
- Real estate + investment integration — natural fit given IMMO lineage
- Broker migration guide — practical, avoids costly surprises

**Defer to v2+:**
- Tax-loss harvesting (requires accurate cost basis + live prices)
- Pension gap projection (actuarial complexity)
- DARF end-to-end automation workflow
- Insurance comparison
- Real-time broker API sync
- Countries beyond Germany and Brazil

### Architecture Approach

Extend the existing pattern exactly. New domains add a command file, one or more specialist agent files, country-scoped reference docs, and optional profile field extensions. No installer changes needed — `bin/install.js` already copies the full tree.

The single most important structural decision: `.finyx/profile.json` is the shared state hub for all agents. Commands read it, select which reference docs to inject, build a structured task prompt, and spawn agents. Agents are stateless — they receive their full context in the task prompt and return Markdown. They never write state. Commands write state.

**Major components:**
1. `commands/fin/` — user-facing slash-commands; orchestrate workflows, spawn agents, read/write state
2. `agents/finyx-[domain]-[country]-agent.md` — stateless specialist sub-agents; receive context in prompt, return structured Markdown
3. `finyx/references/<country>/` — versioned, domain-scoped knowledge docs injected per-command; one file per domain per country
4. `.finyx/profile.json` — runtime user financial profile; single source of truth; written by `/fin:profile`, read by everything
5. `finyx/skills/` — reusable domain knowledge blocks loaded via agent frontmatter

**Key anti-patterns to avoid:**
- God profile command that collects everything upfront — use lazy field collection instead
- Agent-to-agent communication — commands orchestrate, not agents
- Hardcoded tax rates in command files — rates live only in reference docs
- One fat reference file per country — one file per domain per country, inject only what's needed
- npm dependencies for market data — `curl` + WebFetch only

### Critical Pitfalls

1. **Stale tax rules** — Every reference doc must carry `tax_year:` and `valid_until:` metadata. Commands surface a disclaimer when the doc year doesn't match the current year. Create a `TAX_YEAR_CHECKLIST.md` enumerating every annually-changing value. This affects all phases.

2. **Wrong jurisdiction for cross-border users** — A single `country` field is insufficient. The profile must capture: primary tax residency, secondary obligations, expat status, years in current country. Address this in the profile phase — it cannot be retrofitted.

3. **False confidence in AI calculations** — Agent system prompts must explicitly distinguish "orientation advice" from "estimates." Calculations (Vorabpauschale, DARF amounts) must always be flagged as estimates requiring professional verification. Disclaimer blocks are mandatory in every output, not optional.

4. **Multi-broker Freistellungsauftrag miscalculation** — The German tax advisor must capture all broker accounts and enforce that the total Freistellungsauftrag across brokers cannot exceed €1,000 (single) / €2,000 (married). This is a required workflow step, not optional.

5. **Prompt injection via user profile fields** — Free-form profile fields (notes, asset descriptions) must be wrapped in `<user_data>` delimiters in all prompt templates. Structured enums preferred for high-trust fields. Address in the profile phase before any agent reads profile data.

## Implications for Roadmap

Based on combined research, the natural phase structure follows the dependency graph: profile gates everything, Germany is simpler and better-documented than Brazil, investment advisory builds on portfolio data, pension planning requires the most profile context and is highest complexity.

### Phase 1: User Financial Profile + Foundation

**Rationale:** Every specialist command is gated on `.finyx/profile.json`. Nothing else can function without it. This phase also establishes the patterns (disclaimer templates, multi-broker fields, prompt injection mitigations, tax-year metadata conventions) that all subsequent phases inherit.
**Delivers:** `/fin:profile` command, `.finyx/profile.json` schema with cross-border fields, prompt security conventions, disclaimer template, reference doc structure
**Addresses:** User Financial Profile (FEATURES.md table stakes)
**Avoids:** Pitfalls 2 (cross-border), 5 (prompt injection), 10 (advisory positioning)

### Phase 2: German Tax Advisor

**Rationale:** Germany is the better-documented country, with well-established rules and direct continuity from IMMO's existing tax reference content. Validates the full agent pattern (command → profile → reference injection → agent → output) before adding Brazil.
**Delivers:** `/fin:tax` (Germany), `finyx-tax-de-agent`, German tax reference docs (Abgeltungssteuer, Sparerpauschbetrag, Vorabpauschale, Teilfreistellung)
**Addresses:** All German tax table-stakes features
**Avoids:** Pitfalls 1 (stale rules), 4 (multi-broker Freistellungsauftrag), 8 (Vorabpauschale base rate)

### Phase 3: Brazilian Tax Advisor

**Rationale:** Second country validates the multi-country architecture. Brazilian rules are fully researched and must reflect Law 15,270/2025 from the start. Building after Germany means the agent pattern is proven.
**Delivers:** `/fin:tax` (Brazil path), `finyx-tax-br-agent`, Brazilian tax reference docs (IR stocks/ETFs, FII exemptions, DARF, come-cotas, Law 15,270/2025)
**Addresses:** All Brazilian tax table-stakes features
**Avoids:** Pitfalls 6 (FII exemption conditions), 7 (come-cotas liquidity), 11 (Brazil tax reform lag)

### Phase 4: Investment Advisor + Portfolio Tracker

**Rationale:** Core value proposition for investors. Depends on the profile (risk tolerance, holdings) established in Phase 1. Introduces market data access via WebSearch/Finnhub/brapi.dev.
**Delivers:** `/fin:market`, `/fin:portfolio`, `finyx-investment-agent`, portfolio tracker (holdings, allocation, rebalancing)
**Addresses:** Risk profile assessment, ETF recommendations, asset class explanations, portfolio allocation, rebalancing suggestions
**Avoids:** Pitfall 3 (confidence signaling with hallucinated market data)

### Phase 5: Broker Comparison

**Rationale:** Relatively static reference data; low development complexity. Can be built in parallel with Phase 4 but depends on the profile knowing country and asset types. A standalone reference command.
**Delivers:** `/fin:brokers`, broker reference docs (Germany/EU + Brazil), profile-based recommendation logic
**Addresses:** Fee comparison, profile-based recommendation, tax reporting quality note, cross-border account guidance

### Phase 6: German Pension Planning

**Rationale:** Highest profile context dependency (employment status, income, family status, Steuerklasse all required). Building after investment advisory means the profile is well-populated.
**Delivers:** `/fin:pension` (Germany), `finyx-pension-de-agent`, Riester/Rürup/bAV comparison, Zulagen calculator, Rürup deduction estimate
**Addresses:** All German pension table-stakes features
**Avoids:** Pitfall 5 (Riester/Rürup net-benefit oversimplification), Pitfall 12 (bAV portability)

### Phase 7: Brazilian Pension Planning

**Rationale:** Mirrors Phase 6 for Brazil. Requires profile fields for IR regime, investment horizon, and INSS contribution history. INSS expat treatment requires additional research before building.
**Delivers:** `/fin:pension` (Brazil path), `finyx-pension-br-agent`, PGBL vs VGBL decision guide, INSS contribution guidance, progressive vs regressive regime comparison
**Addresses:** All Brazilian pension table-stakes features

### Phase 8: Tax-Aware Investing (Differentiators)

**Rationale:** Builds on portfolio tracker (Phase 4) and tax advisors (Phases 2-3). High-value differentiators: German Vorabpauschale pre-calculation, Sparerpauschbetrag optimization across brokers, Brazilian isenção planning.
**Delivers:** Year-end Vorabpauschale pre-calculation, multi-broker Freistellungsauftrag optimizer, Brazilian R$20k isenção monthly planner, DARF end-to-end workflow

### Phase Ordering Rationale

- Profile before everything else — it is the runtime dependency for all agents
- Germany before Brazil — simpler rules, better-documented, existing IMMO tax content to reuse, faster to validate the pattern
- Investment advisory before pension — portfolio data enriches pension projections
- Broker comparison is a self-contained reference command; can be scheduled flexibly
- Differentiator features (tax-aware investing) require both the portfolio tracker and tax advisors to exist — naturally last

### Research Flags

Phases needing deeper research during planning:
- **Phase 7 (Brazilian Pension):** INSS expat treatment is explicitly flagged as unresolved in PITFALLS.md. Needs targeted research before building the agent.
- **Phase 8 (Tax-Aware Investing):** Tax-loss harvesting requires accurate cost basis handling — edge cases (corporate actions, splits, fund mergers) need scoping before implementation.

Phases with standard patterns (can skip research-phase):
- **Phase 1 (Profile + Foundation):** Fully defined by existing IMMO pattern + pitfall mitigations from research.
- **Phase 2 (German Tax Advisor):** Well-documented rules; reference docs fully specified in STACK.md.
- **Phase 5 (Broker Comparison):** Pure reference data; no algorithmic complexity.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Official docs verified for Finnhub, brapi.dev, Bundesbank SDMX, OpenFIGI. Yahoo Finance/Alpha Vantage exclusions confirmed. |
| Features | MEDIUM-HIGH | German tax ecosystem well-documented. Brazilian rules sourced from Portuguese-language financial press + official sources + Law 15,270/2025 verified via KPMG. |
| Architecture | HIGH | Based on official Claude Code sub-agent docs + direct codebase inspection. Pattern proven by existing IMMO system. |
| Pitfalls | HIGH | Tax pitfalls sourced from official tax authority rules. Prompt injection risk from documented Claude Code CVE. Legal liability from FPA and Kitces. |

**Overall confidence:** HIGH

### Gaps to Address

- **INSS expat treatment:** How does a Brazilian citizen living in Germany contribute to (or exit from) the Brazilian social security system? Not covered in research. Must resolve before Phase 7.
- **Broker fee volatility:** Broker fee comparison data is researcher-assessed as MEDIUM confidence because fees change frequently. Establish a quarterly review process before shipping Phase 5, and date all broker reference docs.
- **Law 15,270/2025 implementation details:** The dividend withholding tax (10% on dividends >BRL 50k/month) and its interaction with FII dividend exemption rules need regulatory source verification — current research draws from KPMG and Mayer Brown secondary sources.
- **Vorabpauschale 2026 Basiszins:** The 2025 Basiszins is confirmed at 2.53%. The 2026 rate will be published by BMF in early 2026. Reference docs must be updated as soon as it is available.

## Sources

### Primary (HIGH confidence)
- Finnhub official docs + npm: https://finnhub.io/docs/api, https://www.npmjs.com/package/finnhub
- brapi.dev official docs: https://brapi.dev/docs
- Bundesbank SDMX API: https://www.bundesbank.de/en/statistics/time-series-databases/help-for-sdmx-web-service
- OpenFIGI API: https://www.openfigi.com/api/documentation
- Claude Code Sub-Agents docs: https://code.claude.com/docs/en/sub-agents
- Existing IMMO codebase (direct inspection)
- BaFin Riester pensions: https://www.bafin.de/EN/Verbraucher/Altersvorsorge/Riester/riester_node_en.html

### Secondary (MEDIUM confidence)
- German Vorabpauschale 2025/2026: https://quantroutine.com/learn/investing-taxes-germany/, https://hexn.io/...
- Brazilian IR investments 2026: https://www.idinheiro.com.br/investimentos/como-declarar-investimentos-no-imposto-de-renda/
- KPMG: Brazil Law 15,270/2025: https://kpmg.com/xx/en/our-insights/gms-flash-alert/flash-alert-2025-251.html
- Mayer Brown: Law 15,270/2025: https://www.mayerbrown.com/en/insights/publications/2025/12/...
- Broker comparisons (DE/BR): perfinex.de, northern.finance, rankia.com.br

### Tertiary (MEDIUM-LOW confidence)
- Broker fee tables — change frequently; verified as of Q1 2026 but subject to drift
- Brazilian FII exemption conditions — cross-referenced Gringo Investor + official InvStG equivalents; recommend official Receita Federal confirmation

---
*Research completed: 2026-04-06*
*Ready for roadmap: yes*
