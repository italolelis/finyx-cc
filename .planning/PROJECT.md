# Finyx

## What This Is

Finyx is an open-source personal finance advisor built as a collection of Claude Code slash-commands and specialist AI agents. It evolves the existing IMMO real estate analysis tool into a comprehensive financial planning system — covering tax optimization, investment portfolio management, insurance comparison, pension planning, and real estate investment. Users interact through a single CLI interface (`/fin:*` commands) while specialist agents handle domain-specific analysis backed by shared user context and memory.

## Core Value

A single AI-powered financial advisor that knows your full financial picture — tax situation, investments, insurance, pensions, and real estate — and gives integrated, country-aware advice rather than siloed recommendations.

## Requirements

### Validated

- ✓ Real estate investment analysis pipeline (scout, analyze, filter, compare, stress-test, report) — existing
- ✓ German tax rules for real estate (AfA, Grunderwerbsteuer, marginal rate calculations) — existing
- ✓ Investor profile with interactive questionnaire — existing
- ✓ Location research with transport/Erbpacht assessment — existing
- ✓ Multi-format report generation (Markdown, PDF, short/full) — existing
- ✓ npm distribution via `npx immo-cc` with global/local install — existing

### Active

- [ ] User financial profile — structured interview covering income, tax class, family status, goals, risk tolerance, country (Germany/Brazil)
- [ ] German tax advisor — tax class explanation, optimization opportunities, deduction guidance, return filing tips
- [ ] Brazilian tax advisor — basic tax rules for investments and real estate for Brazilian residents/expats
- [ ] Investment advisor — stock, ETF, futures analysis with portfolio balancing recommendations
- [ ] Portfolio tracker — current holdings analysis, rebalancing suggestions, sector/geography allocation
- [ ] Market intelligence — live market news, stock/ETF data via APIs + web search fallback
- [ ] Broker comparison — German/European (Trade Republic, Scalable, ING, comdirect), US (Interactive Brokers, Charles Schwab), Brazilian (XP, NuInvest, BTG) broker recommendations based on user profile
- [ ] German tax-aware investing — Sparerpauschbetrag, Vorabpauschale, Teilfreistellung, tax-loss harvesting guidance
- [ ] Brazilian tax-aware investing — IR rules for stocks/ETFs/FIIs, come-cotas, DARF obligations
- [ ] Pension planning — Riester, Rürup, bAV analysis for Germany; INSS/previdência privada for Brazil
- [ ] Shared memory system — global user profile accessible across all specialist agents
- [ ] Real estate expansion — Brazil market support (property analysis, tax rules, rental yields)
- [ ] Project rename — rebrand from immo-cc to finyx across package, commands, agents, docs

### Out of Scope

- Insurance comparison (check24-style) — deferred to v2, high complexity and data sourcing challenges
- Automated trading/execution — legal liability, this is advisory only
- Tax return filing automation — too risky, advisor guides but user files
- Real-time portfolio sync with brokers — API complexity, defer to v2
- Mobile or web app — CLI-first, other interfaces later
- Countries beyond Germany and Brazil — expand after v1 validates the multi-country pattern

## Context

- **Existing codebase:** Mature real estate analysis tool (IMMO) with slash-command architecture, 3 specialist agents, country-specific reference docs, npm distribution
- **Architecture pattern:** Meta-prompting system — all logic in Markdown prompts, single Node.js installer, agents spawned via Claude Code's Task tool
- **Extension model:** New financial domains follow same pattern — reference docs for domain knowledge, command `.md` files for workflows, agent `.md` files for specialists
- **Country structure:** `immo/references/germany/` pattern extends to `finyx/references/brazil/` for multi-country support
- **Data sources:** Financial APIs (Yahoo Finance, etc.) where available, web search as fallback for market data and news
- **Target audience:** Global — anyone managing finances across Germany and Brazil, with architecture ready for more countries
- **Open source:** MIT license, community-driven, designed for contributors to add country modules

## Constraints

- **Tech stack**: Claude Code slash-command architecture — no application framework, all logic in Markdown prompts
- **Runtime**: Node.js >=16.7.0, npm distribution, zero runtime dependencies
- **Legal**: Advisory only — all recommendations include disclaimers, no automated execution
- **Data freshness**: Market data via live web search + APIs, tax rules in versioned reference docs updated per tax year
- **Country scope**: Germany + Brazil for v1, architecture must support adding countries without refactoring

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Rename to Finyx | Broader scope beyond real estate needs a new brand | — Pending |
| Germany + Brazil v1 | User has direct experience in both markets | — Pending |
| CLI skills architecture | Proven pattern from IMMO, low friction, extensible | — Pending |
| Advisory only, no execution | Legal risk of automated trading/filing too high for open source | — Pending |
| APIs + web search for data | APIs for structured data, web search for news/freshness | — Pending |
| Insurance deferred to v2 | Data sourcing complexity, focus v1 on tax + investments | — Pending |
| Shared user profile | All agents read same profile — integrated advice requires shared context | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? Move to Out of Scope with reason
2. Requirements validated? Move to Validated with phase reference
3. New requirements emerged? Add to Active
4. Decisions to log? Add to Key Decisions
5. "What This Is" still accurate? Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-06 after initialization*
