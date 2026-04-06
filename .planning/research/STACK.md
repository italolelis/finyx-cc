# Technology Stack

**Project:** Finyx — Personal Finance Advisor CLI
**Milestone:** Adding financial advisory capabilities (tax, investment, pension) to existing IMMO slash-command architecture
**Researched:** 2026-04-06

---

## Architecture Constraint

This is NOT a traditional application stack. The existing system is a **meta-prompting architecture**: all logic lives in Markdown prompts, a single Node.js installer handles distribution, and agents are spawned via Claude Code's Task tool. New financial domains extend this pattern exactly — no application framework, no new runtime, no dependency additions.

The "stack" question for this project is therefore: **what external data sources and reference materials feed the agents?**

---

## Market Data APIs

### Primary: Finnhub

| Property | Value |
|----------|-------|
| URL | https://finnhub.io |
| Free tier | 60 API calls/minute, no credit card required |
| Coverage | 60+ exchanges, US real-time, international 15-min delay |
| ETF support | Global ETF holdings data |
| Node.js SDK | Official: `npm install finnhub` (v2.0.13, maintained) |
| Confidence | HIGH — verified against official docs and npm |

**Why Finnhub:** The only major financial API with an official, maintained Node.js SDK, free tier with no rate daily cap (60/min is sufficient for advisory use, not HFT), and institutional-grade data including earnings surprises and insider sentiment relevant to investment advice. Covers both US and European markets on the free tier.

**What Finnhub does NOT cover well:** Brazilian B3 stocks/FIIs. Use brapi.dev for that.

**Access pattern for slash-command architecture:** Agents invoke via `curl` or `node -e` inline scripts in command prompts, or instruct the user to set `FINNHUB_API_KEY` in their environment. The key is user-supplied — Finyx never bundles credentials.

### Brazil-specific: brapi.dev

| Property | Value |
|----------|-------|
| URL | https://brapi.dev |
| Free tier | Unlimited queries for popular B3 assets (PETR4, VALE3, etc.) |
| Coverage | B3 stocks, FIIs, BDRs, ETFs, IBOVESPA, economic indicators |
| Node.js | REST API, no official SDK — direct HTTP calls |
| Confidence | HIGH — official docs verified, active in 2025/2026 |

**Why brapi.dev:** Only purpose-built free API for B3 data with FII support. Finnhub covers Brazilian stocks via ticker but lacks FII-specific data (dividend yields, patrimony, P/VP ratios) which are critical for Brazilian real estate investment fund analysis.

**No API key required** for basic usage. Paid plans unlock higher rate limits and historical depth.

### Symbol Mapping: OpenFIGI

| Property | Value |
|----------|-------|
| URL | https://www.openfigi.com |
| Free tier | Free, no daily/monthly limit (rate-limited without key) |
| Purpose | Map ISINs to tradeable tickers across exchanges |
| Confidence | HIGH — official Bloomberg-backed open standard |

**Why needed:** European ETF positions (e.g., MSCI World accumulating ETFs traded on XETRA) use ISIN identifiers, not Yahoo-style tickers. OpenFIGI resolves ISINs to exchange-specific symbols that Finnhub can query.

**Not needed for MVP.** Include in portfolio tracker phase when users input holdings by ISIN.

---

## Tax Rule Data Sources

### German Tax Rules: Static Reference Docs + Official Sources

There is **no tax rule API** for Germany. This is by design: the agent pattern uses versioned reference Markdown files (already proven in IMMO for AfA, Grunderwerbsteuer). The same pattern applies here.

| Source | Data | Update frequency | Confidence |
|--------|------|-----------------|------------|
| Bundesministerium der Finanzen (BMF) | Sparerpauschbetrag, Abgeltungssteuer rates, Teilfreistellung | Annual | HIGH |
| Bundesbank SDMX API | Basiszins (for Vorabpauschale calculation) | Semi-annual | HIGH |
| §18 InvStG (Investmentsteuergesetz) | Vorabpauschale formula | Annual | HIGH |

**Bundesbank SDMX API endpoint (free, no auth required):**
`https://api.statistiken.bundesbank.de/rest/data/BBK/...`

Agents can call this endpoint to retrieve the current Basiszins programmatically. The 2025 Basiszins is 2.53% (Ministry of Finance), with the Bundesbank civil code Basiszins (§247 BGB) at 2.27% (Jan 2025) and 1.27% (Jul 2025) — these are different rates used for different calculations.

**Key German tax parameters to encode in reference docs:**
- Abgeltungssteuer: 25% flat + Soli (5.5% of tax) + Kirchensteuer (8-9% if applicable)
- Sparerpauschbetrag: EUR 1,000 (single) / EUR 2,000 (married)
- Teilfreistellung: 30% for equity ETFs, 15% for mixed funds, 60% for real estate funds
- Vorabpauschale formula: `Basisertrag = Rücknahmepreis_Jan1 × 0.7 × Basiszins_BMF`

### Brazilian Tax Rules: Static Reference Docs

No developer API exists for Brazilian tax rules. Same pattern: versioned reference Markdown files updated per tax year.

| Topic | Key rules to encode | Confidence |
|-------|-------------------|------------|
| IR on stocks | Day-trade: 20%, Swing-trade exempt up to R$20k/month sales | HIGH |
| IR on ETFs | 15% on gains (no monthly exemption, unlike stocks) | HIGH |
| FII dividends | Exempt for individuals if FII listed on exchange with 50+ shareholders | HIGH |
| FII capital gains | 20% for all individuals | HIGH |
| Come-cotas | Applies to funds (not FIIs): semi-annual IRRF in May and Nov | HIGH |
| DARF | Self-assessed, due by last business day of following month | HIGH |
| PGBL | Contributions deductible up to 12% of taxable income; tax on total withdrawal | HIGH |
| VGBL | No deduction; tax only on earnings at withdrawal | HIGH |
| PGBL/VGBL regimes | Progressive vs regressive — since Law 14,803/2024, choosable at withdrawal | HIGH |
| IOF on PGBL/VGBL | 5% on contributions >R$300k/CPF at same insurer (2025); >R$600k/year from 2026 | MEDIUM |

**Note on Law 15,270/2025 (enacted Dec 2025):** Introduces dividend taxation and a minimum IRPFM for incomes >R$600k/year. This is a significant change. Reference docs must be versioned and this law's effective date (largely 2026) noted clearly.

---

## Pension Planning

### Germany: Static Reference + Calculator Logic in Prompts

No open-source German pension API exists. Calculations are deterministic given inputs — they belong in agent prompt logic, not external APIs.

| Product | Key calculation inputs to encode |
|---------|----------------------------------|
| Riester | Eigenbeitrag (4% gross income), Grundzulage (EUR 175/yr), Kinderzulage (EUR 185-300/yr), tax deduction ceiling (EUR 2,100) |
| Rürup (Basis-Rente) | Max deductible 2025: EUR 29,344 (single) / EUR 58,688 (married), 100% deductible |
| bAV (Direktversicherung) | Steuer- und SV-freier Anteil: 4% × BBG (Beitragsbemessungsgrenze) |
| GRV (Gesetzliche RV) | Beitragssatz 18.6%, hälftig (employer/employee), Rentenformel |

**BBG West 2025:** EUR 90,600/year — needed for bAV calculations.

### Brazil: Static Reference + Calculator Logic

| Product | Key rules to encode |
|---------|-------------------|
| INSS | Contribuição: 7.5–14% depending on salary bracket (tabela progressiva) |
| PGBL vs VGBL choice | PGBL for formal employees using IRPF simplified deduction → wrong; PGBL only if filing full deductions |
| Fator previdenciário | Formula: Tc × a × (1 + (Id + Tc × a) / 100) — complex, encode as calculator |
| Previdência privada regimes | Regressivo: 35% at year 0, down to 10% after 10 years |

---

## Broker Comparison

No API. This is static reference data maintained as structured Markdown tables, updated manually when broker pricing changes. The comparison matrix covers:

| Market | Brokers |
|--------|---------|
| Germany/EU | Trade Republic, Scalable Capital, ING, comdirect, DKB |
| US | Interactive Brokers, Charles Schwab |
| Brazil | XP Investimentos, NuInvest, BTG Pactual, Rico |

**Data points per broker to maintain:** custody fee, trade fee (ETF, stock, options), available markets, tax reporting quality, API/CSV export support, minimum account.

**Confidence:** MEDIUM — broker fees change frequently, requires human review quarterly.

---

## Data Access Pattern for Slash-Command Architecture

Since all logic is Markdown prompts with no runtime dependencies:

| Data type | Access method | Key decision |
|-----------|--------------|--------------|
| Live stock/ETF quotes | Agent instructs Claude to use `WebFetch` or `Bash(curl)` against Finnhub REST | No SDK needed — raw REST via curl |
| B3/FII quotes | Agent uses `Bash(curl)` against brapi.dev REST | No auth needed for free tier |
| Bundesbank Basiszins | Agent uses `Bash(curl)` against Bundesbank SDMX API | Free, no auth, JSON/XML output |
| Tax rules | Embedded in `.claude/finyx/references/{country}/tax-*.md` | Updated annually by maintainers |
| Pension parameters | Embedded in `.claude/finyx/references/{country}/pension-*.md` | Updated annually |
| Broker data | Embedded in `.claude/finyx/references/brokers/*.md` | Reviewed quarterly |
| Market news | Claude's WebSearch tool directly | No API key required |
| ISIN mapping | Agent uses `Bash(curl)` against OpenFIGI REST | Free, no auth for basic usage |

**API key management:** Agents prompt users to set environment variables (`FINNHUB_API_KEY`, optionally `BRAPI_TOKEN` for higher limits). Keys stored in user's shell environment, never in the repository.

---

## What NOT to Use

| Option | Why not |
|--------|---------|
| Yahoo Finance (unofficial) | No official API, scrapes Yahoo endpoints, violates ToS, breaks regularly — confirmed by multiple sources |
| Alpha Vantage | Free tier degraded (25 req/day confirmed in some sources vs 500 claimed in others — contradictory), unreliable for production |
| yfinance Python library | Python-specific, not usable in this Node.js/Markdown architecture |
| Financial Modeling Prep (FMP) | US-focused, weak EU/Brazilian coverage, paid for most useful endpoints |
| Any server-side financial middleware | Contradicts the zero-runtime-dependency constraint |
| npm financial calculation libraries | Adds dependency surface area; arithmetic in prompts or inline node -e is sufficient for advisory calculations |

---

## Reference File Structure

Following the existing IMMO pattern, extend to:

```
.claude/finyx/references/
  germany/
    tax-capital-gains-{year}.md      # Abgeltungssteuer, Sparerpauschbetrag
    tax-etf-{year}.md                # Vorabpauschale, Teilfreistellung
    tax-real-estate-{year}.md        # Existing AfA, GrESt (already exists in IMMO)
    pension-state-{year}.md          # GRV, BBG, Rentenformel
    pension-riester-{year}.md        # Zulagen, Grenzen
    pension-ruerup-{year}.md         # Absetzbarkeit, Maximalbeträge
    pension-bav-{year}.md            # bAV vehicles, steuerfreie Beträge
    brokers-germany-eu-{year}.md     # TR, Scalable, ING, comdirect
  brazil/
    tax-stocks-{year}.md             # IR stocks/ETFs, DARF, exemptions
    tax-fii-{year}.md                # FII dividends, capital gains
    tax-previdencia-{year}.md        # PGBL/VGBL, IOF, Law 15270/2025
    pension-inss-{year}.md           # Tabela INSS, fator previdenciário
    pension-private-{year}.md        # PGBL vs VGBL decision framework
    brokers-brazil-{year}.md         # XP, NuInvest, BTG, Rico
  global/
    brokers-us-{year}.md             # IBKR, Schwab
```

---

## Sources

- Finnhub official docs: https://finnhub.io/docs/api
- Finnhub npm package: https://www.npmjs.com/package/finnhub
- brapi.dev docs: https://brapi.dev/docs
- OpenFIGI API: https://www.openfigi.com/api/documentation
- Bundesbank SDMX API: https://www.bundesbank.de/en/statistics/time-series-databases/help-for-sdmx-web-service
- Bundesbank Basiszins 2025: https://www.bundesbank.de/en/press/press-releases/announcement-of-the-basic-rate-of-interest-as-of-1-january-2025-adjustment-to-2-27--948660
- ECB Data Portal API: https://data.ecb.europa.eu/help/api/data
- German Vorabpauschale 2025 (Basiszins 2.53%): https://quantroutine.com/learn/investing-taxes-germany/
- Brazilian Law 15,270/2025 (dividend tax reform): https://www.mayerbrown.com/en/insights/publications/2025/12/enactment-of-law-no-15270-2025-which-establishes-dividend-taxation-expands-the-exemption-threshold-and-introduces-a-minimum-tax-on-high-incomes
- Brazilian PGBL/VGBL IOF changes 2025: https://investnews.com.br/investimentos/pgbl-vgbl-previdencia-privada/
- Free API comparison 2025: https://noteapiconnector.com/best-free-finance-apis
- Twelve Data review: https://medium.com/coinmonks/the-7-best-financial-apis-for-investors-and-developers-in-2025-in-depth-analysis-and-comparison-adbc22024f68
