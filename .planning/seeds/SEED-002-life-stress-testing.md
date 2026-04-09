---
id: SEED-002
status: dormant
planted: 2026-04-09
planted_during: v1.2 Health Insurance Advisor
trigger_when: next milestone (v1.3)
scope: Medium
---

# SEED-002: Comprehensive life stress testing engine

## Why This Matters

The existing `/finyx:stress-test` only covers real estate scenarios (zero appreciation, interest rate hikes). A personal finance advisor should stress-test the ENTIRE financial picture — job loss, health crises, market crashes, geopolitical events, family changes. These tests should feed into `/finyx:insights` as input for recommendations.

The user wants tests that are grounded in CURRENT events (e.g., Iran conflict impact on energy stocks, tariff wars on EU exports) via live web search, not generic hypotheticals.

**Why:** Real financial resilience comes from understanding how your full picture reacts to shocks, not just one asset class. This is the difference between a spreadsheet and a real advisor.

**How to apply:** Either evolve `/finyx:stress-test` into a generic command or create a new `/finyx:stress` command. Use a research agent to pull current events and model their financial impact on the user's specific portfolio, insurance, employment, and real estate.

## When to Surface

**Trigger:** Next milestone (v1.3)

This seed should be presented during `/gsd:new-milestone` when the milestone scope matches any of these conditions:
- When stress testing or risk analysis features are planned
- When insights/recommendations improvements are a goal
- When real-time market/news integration is discussed
- When portfolio risk management features are scoped

## Scope Estimate

**Medium** — 1-2 phases. Phase 1: stress test engine (scenarios + calc). Phase 2: insights integration + current events research agent.

## Stress Test Categories

### Employment & Income
- **Job loss** — "What if I lose my job tomorrow?" Emergency fund runway, unemployment benefits (ALG I for DE), severance, how long before forced asset liquidation
- **Income reduction** — 20% salary cut scenario. Impact on savings rate, investment contributions, loan servability
- **Career change** — Moving from employee to self-employed. Insurance implications (must choose PKV/GKV), pension gap, tax class change

### Market & Investment
- **Market crash** — 30% equity drawdown. Impact on net worth, portfolio recovery timeline, rebalancing opportunities
- **Interest rate spike** — +2% on variable-rate debt. Mortgage refinancing cost, bond portfolio impact
- **Currency shock** — EUR/BRL 20% move. Impact for cross-border users with Brazilian investments
- **Sector concentration** — "What if tech stocks drop 40%?" Portfolio-specific sector analysis

### Geopolitical & Current Events (LIVE SEARCH)
- **Active conflicts** — Research agent searches current news, models energy price impact on portfolio, supply chain exposure
- **Trade wars / tariffs** — Impact on EU-exposed ETFs, emerging market funds
- **Sanctions** — Exposure to sanctioned-country assets or counterparties
- **Regulatory changes** — Upcoming tax law changes, pension reform proposals

### Health & Insurance
- **Critical illness** — Long-term disability scenario. Insurance coverage gaps, savings depletion timeline
- **PKV premium shock** — 15% annual increase for 3 consecutive years. Affordability cliff, Basistarif fallback
- **Family health event** — Partner or child requiring extended care. Impact on dual-income household

### Real Estate (extends existing)
- **Zero appreciation** — Already exists, keep
- **Interest rate hike at refinancing** — Already exists, keep
- **Vacancy** — Rental property empty for 6 months. Cash flow impact
- **Tenant default** — No rent for 3 months + legal costs

### Family & Life Events
- **Divorce** — Asset split, Versorgungsausgleich (pension equalization), housing cost doubling
- **New child** — Reduced income (Elterngeld), Kindergeld, insurance impact (Familienversicherung in GKV vs PKV per-child cost)
- **Partner stops working** — From dual to single income. Tax class optimization (III/V), Familienversicherung eligibility

## Integration with /finyx:insights

Each stress test should produce:
1. **Impact score** — How severely this scenario affects the user (low/medium/high/critical)
2. **Time to crisis** — How many months of runway before the scenario becomes critical
3. **Mitigation actions** — Concrete steps to reduce exposure (with € cost/benefit)
4. **Recommendation** — Feed top-3 highest-risk scenarios into insights as "Risk Alerts"

## Breadcrumbs

Related code and decisions in the current codebase:

- `commands/finyx/stress-test.md` — Existing real estate stress test (to be evolved or replaced)
- `commands/finyx/insights.md` — Should consume stress test output for recommendations
- `agents/finyx-projection-agent.md` — Already does net worth projection with scenarios, could be extended
- `agents/finyx-allocation-agent.md` — Emergency fund check already exists, stress tests would feed into it
- `finyx/references/insights/benchmarks.md` — Emergency fund threshold (6 months) used by allocation agent
- `agents/finyx-insurance-calc-agent.md` — PKV projection scenarios already model premium growth
- `.planning/PROJECT.md` — Core Value mentions "integrated, country-aware advice" — stress testing across domains is the ultimate expression of this

## Notes

- User explicitly wants CURRENT EVENT integration — agents must WebSearch for news and model impact, not just use static hypotheticals
- The "what if I lose my job" test is the single most requested personal finance scenario globally — table stakes
- Geopolitical tests need a research agent that can assess portfolio exposure to specific events (e.g., "you hold MSCI EM which has 15% China exposure — tariff escalation risk is MEDIUM")
- Consider a `/finyx:stress` command (generic) that subsumes the real estate `/finyx:stress-test` while adding all other categories
- Each test should respect the user's country context (ALG I for DE, FGTS/seguro-desemprego for BR)
- Tests should compose: "job loss + market crash" is a realistic combined scenario that's worse than either alone
