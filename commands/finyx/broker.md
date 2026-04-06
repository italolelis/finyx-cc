---
name: finyx:broker
description: Broker comparison — fee analysis and profile-based recommendation for German and Brazilian brokers
allowed-tools:
  - Read
  - Bash
  - AskUserQuestion
---

<objective>

Deliver a profile-based broker recommendation with fee comparisons and tax reporting quality differences.

This command:
1. Reads `.finyx/profile.json` and detects which countries are active
2. Checks broker reference doc staleness (warns if older than 6 months)
3. For Germany: presents fee comparison for Trade Republic, Scalable Capital, ING, and comdirect
4. For Brazil: presents fee comparison for NuInvest, XP Investimentos, and BTG Pactual
5. Collects trading frequency and strategy via AskUserQuestion, then applies the decision matrix to recommend a broker
6. Explains tax reporting quality differences between German and foreign brokers

Output is conversational advisory text — no files are written. All guidance includes the legal disclaimer.

</objective>

<execution_context>

@~/.claude/finyx/references/disclaimer.md
@~/.claude/finyx/references/germany/brokers.md
@~/.claude/finyx/references/brazil/brokers.md
@.finyx/profile.json

</execution_context>

<process>

## Phase 1: Validation

**Check profile exists:**
```bash
[ -f .finyx/profile.json ] || { echo "ERROR: No financial profile found. Run /finyx:profile first to set up your profile."; exit 1; }
```

**Read `.finyx/profile.json`** and extract:
- `identity.cross_border` — cross-border flag
- `identity.family_status` — "single" or "married"
- `countries.germany.tax_class` — null means Germany not active
- `countries.brazil.ir_regime` — null means Brazil not active

**Determine active countries:**
- Germany active if: `countries.germany.tax_class != null`
- Brazil active if: `countries.brazil.ir_regime != null`

**If neither country is active:**
```
ERROR: No country data found in your profile.

Run /finyx:profile to complete the country-specific sections
(German tax class or Brazilian IR regime must be set).
```
Stop here.

## Phase 2: Reference Doc Staleness Check

For each broker reference doc loaded (`germany/brokers.md` and `brazil/brokers.md`), extract `last_verified` from the YAML frontmatter and check if it is more than 6 months old:

```bash
node -e "
  const last = new Date('LAST_VERIFIED_DATE');
  const now = new Date();
  const days = (now - last) / 86400000;
  if (days > 180) console.log('STALE');
  else console.log('OK');
"
```

If the document is STALE, emit this warning banner before any broker comparison output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 FINYX ► BROKER: STALENESS WARNING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Broker reference data was last verified on [LAST_VERIFIED_DATE].
Broker fees and product offerings change frequently.
Verify all fee details against the broker websites listed
in this output before making any decisions.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Continue regardless — data is still useful even when flagged stale.

## Phase 3: German Broker Comparison

*Execute this phase only if Germany is active.*

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 FINYX ► BROKER: GERMAN BROKERS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Using the data from `germany/brokers.md`, present the fee comparison table:

| Broker | Trade Fee | Sparplan Fee | Custody | Regulated By |
|--------|-----------|--------------|---------|--------------|
| Trade Republic | €1.00/order | Free | None | BaFin |
| Scalable Capital FREE | Free ≥€250 / €0.99 below | Free | None | BaFin |
| Scalable Capital PRIME+ | €4.99/month flat rate | Free | None | BaFin |
| ING | €4.90 + 0.25% | 1.75% (promo: free) | None | BaFin |
| comdirect | €4.90 + 0.25% | 1.5% (promo: free) | None | BaFin |

**Key differentiators:**
- **Trade Republic** — cheapest per-trade at €1.00; free Sparplans; best for ETF-first or infrequent buy-and-hold investors
  - URL: https://traderepublic.com
- **Scalable Capital PRIME+** — best for high-volume active traders (flat €4.99/month pays off above ~5 trades/month); all Sparplans free
  - URL: https://de.scalable.capital
- **ING** — full banking + brokerage integration; better for existing ING customers; not cost-competitive for frequent trading
  - URL: https://ing.de
- **comdirect** — full-service bank with brokerage; higher fees than neo-brokers; custody fee applies if no active Sparplan or less than 2 trades per quarter
  - URL: https://comdirect.de

**Note:** All German brokers support Freistellungsauftrag and generate Jahressteuerbescheinigung automatically — important for tax compliance (see Phase 6).

## Phase 4: Brazilian Broker Comparison

*Execute this phase only if Brazil is active.*

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 FINYX ► BROKER: BRAZILIAN BROKERS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Using the data from `brazil/brokers.md`, present the fee comparison table:

| Broker | Corretagem (App) | Custódia | Strengths |
|--------|-----------------|----------|-----------|
| NuInvest | R$0 for ações, ETFs, FIIs, BDRs | None | Zero cost, lowest friction |
| XP Investimentos | Zero for some / varies | None | Wide renda fixa selection, research tools |
| BTG Pactual | R$4.50/order (tiered) | None | Institutional renda fixa rates, high-volume tiered pricing |

**Key differentiators:**
- **NuInvest** — zero corretagem via app for all renda variável products (ações, ETFs, FIIs, BDRs, options); best entry point for B3 investors
  - URL: https://nuinvest.com.br
- **XP Investimentos** — extensive renda fixa selection (CDB, LCI, LCA, debentures, Tesouro Direto) plus renda variável; full-service platform with research tools; corretagem may vary by product
  - URL: https://xpi.com.br
- **BTG Pactual** — R$4.50/order base rate with volume tiering; strong institutional fixed income desk with CDBs up to 112% CDI; best for higher-volume traders prioritizing fixed income quality
  - URL: https://investimentos.btgpactual.com

**Important note:** B3/CBLC emolumentos (exchange fees) are charged separately by B3 for all renda variável trades — these are NOT set by the broker and apply uniformly regardless of which broker you use.

**DARF reminder:** No Brazilian broker auto-withholds DARF for renda variável capital gains. You are fully responsible for calculating and paying DARF by the last business day of the following month.

## Phase 5: Profile-Based Recommendation

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 FINYX ► BROKER: RECOMMENDATION FOR YOUR PROFILE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Use AskUserQuestion to collect the following (if not already inferable from profile):

**Question 1 — Trading frequency:**
```
How often do you trade?
1. Rarely (1–2 orders per month) — buy-and-hold or Sparplan
2. Regularly (weekly) — moderate active trading
3. Actively (daily or multiple times per week) — frequent trading
```

**Question 2 — Primary strategy:**
```
What is your primary investment approach?
1. ETF savings plan (Sparplan) — automated periodic purchases
2. Buy-and-hold — occasional purchases, long holding periods
3. Active trading — frequent buys and sells based on market moves
```

**Question 3 — Tax simplicity preference:**
```
How important is automatic tax handling to you?
1. Very important — I want my broker to handle withholding and reporting automatically
2. Nice-to-have — I prefer auto-handling but can manage it myself if needed
3. Not important — I am comfortable handling tax reporting manually
```

**Decision matrix — Germany active users:**

| Scenario | Recommended Broker |
|----------|--------------------|
| Sparplan + rarely + tax important | Trade Republic |
| Sparplan + any + wants full banking | ING |
| Active + regularly/daily + tax important | Scalable Capital PRIME+ |
| Buy-and-hold + rarely + cost-sensitive | Trade Republic |
| Active + cost-sensitive + willing to handle tax | Consider Trading212 or IBKR — but warn: manual Anlage KAP + Anlage KAP-INV required (see Phase 6) |

**Decision matrix — Brazil active users:**

| Scenario | Recommended Broker |
|----------|--------------------|
| Sparplan/periodic + rarely + lowest friction | NuInvest |
| Renda fixa priority + any frequency | XP Investimentos or BTG Pactual |
| Active trading + high volume | BTG Pactual (tiered pricing benefits) |
| ETF or FII focus + cost-sensitive | NuInvest (zero corretagem via app) |

**Output format for recommendation:**

```
Recommended broker: [name]

Why this fits your profile:
- [Reason 1 — relates to their trading frequency answer]
- [Reason 2 — relates to their strategy answer]
- [Reason 3 — relates to tax preference answer]

Choose [broker] if:
- [Condition 1]
- [Condition 2]

Avoid [broker] if:
- [Condition 1 — e.g., you trade very actively and the per-order fee compounds]
- [Condition 2]
```

## Phase 6: Tax Reporting Quality

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 FINYX ► BROKER: TAX REPORTING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### German tax reporting quality — German vs foreign broker

*Show this section only if Germany is active.*

| Tax Obligation | German Broker | Foreign Broker (Trading212, IBKR) |
|----------------|---------------|-----------------------------------|
| Abgeltungssteuer withholding | Automatic (26.375% at source) | None — investor self-reports |
| Freistellungsauftrag | Supported — Sparerpauschbetrag assigned | Not possible — non-German entity |
| Jahressteuerbescheinigung | Generated automatically | Not available |
| Vorabpauschale handling | Auto-deducted in January for accumulating ETFs | Investor calculates and declares manually on Anlage KAP-INV |

**Key recommendation:**
> If tax simplicity matters, use a German broker as your primary. If you also use a foreign broker (Trading212, IBKR), allocate your full Sparerpauschbetrag (Freistellungsauftrag) to your German broker(s) — you cannot assign it to foreign brokers.

**Foreign broker compliance overhead:**
Using a foreign broker as your sole German broker means you must:
1. Track all capital gains and dividends manually throughout the year
2. File Anlage KAP and Anlage KAP-INV every year regardless of gain level
3. Calculate Vorabpauschale manually for any accumulating ETFs held at the foreign broker
4. Verify no double-taxation with any foreign withholding credits

### Brazilian tax reporting quality

*Show this section only if Brazil is active.*

> All Brazilian brokers require self-reported DARF for renda variável capital gains. Unlike the German system, no Brazilian broker auto-withholds tax on stock, FII, or ETF capital gains.

Tax obligations are identical regardless of which Brazilian broker you use:
- **DARF (renda variável):** self-calculated and self-paid monthly — code 6015 for stocks/FIIs, code 3317 for day-trade
- **Informe de Rendimentos:** provided by all brokers in February/March — covers dividends and income received, NOT capital gains
- **Annual DIRPF:** you declare bens e direitos, rendimentos isentos (FII dividends), and rendimentos tributáveis (renda fixa interest)

There is no tax-handling quality difference between Brazilian brokers — your compliance obligations are the same regardless of which one you choose.

## Phase 7: Disclaimer

Append the full legal disclaimer from the loaded `disclaimer.md` reference at the end of all advisory output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 LEGAL DISCLAIMER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

[Output the full disclaimer.md content here]

</process>

<error_handling>

**No profile found:**
```
ERROR: No financial profile found.
Run /finyx:profile first to complete your financial profile.
```

**No country data in profile:**
```
ERROR: No country data found in your profile.
Your profile exists but neither Germany (tax_class) nor Brazil (ir_regime) is configured.
Run /finyx:profile to complete the country-specific sections.
```

</error_handling>

<notes>

## Broker Fees Change Frequently

Broker fees are verified at a point in time (see `last_verified` in the reference docs). The staleness check in Phase 2 warns automatically if the reference data is more than 6 months old. Always verify current fees on the broker's website before opening an account or changing brokers. Each broker's URL is provided in the comparison output.

## Recommendation Is Profile-Based, Not Exhaustive

The recommendation in Phase 5 considers trading frequency, investment strategy, and tax simplicity preference. It cannot account for all personal preferences (e.g., UI/UX preference, crypto access, fractional shares, specific ETF availability). Use it as a starting point, not a final decision.

## Country Routing

- Germany-only user: sees Phase 3 (DE comparison) + Phase 5 (recommendation) + Phase 6 (DE tax reporting only)
- Brazil-only user: sees Phase 4 (BR comparison) + Phase 5 (recommendation) + Phase 6 (BR tax note only)
- Cross-border user: sees Phase 3 + Phase 4 + Phase 5 + Phase 6 (both sections)

## Reference Data Staleness

The staleness check uses `node -e` for cross-platform date arithmetic (avoids GNU `date -d` vs macOS `date -j` incompatibility). The threshold is 180 days (approximately 6 months).

</notes>
