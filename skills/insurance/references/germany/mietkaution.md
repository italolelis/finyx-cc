---
tax_year: 2025
country: germany
domain: mietkaution
last_updated: 2026-04-12
source: "§551 BGB (Mietkautionsbegrenzung), Verbraucherzentrale, GDV, VVG"
---

# Mietkautionsversicherung (Rental Deposit Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. §551 BGB sets the maximum deposit at 3× net cold rent — this statutory cap is unchanged. Annual premium rates are market-driven.

---

## Overview

Mietkautionsversicherung replaces the traditional cash security deposit (Mietkaution) that tenants must pay to landlords when renting an apartment. Instead of tying up 2–3 months of net cold rent in a deposit account, the tenant pays an annual premium and the insurer guarantees the deposit amount to the landlord.

- **Who needs it:** Tenants who prefer to keep their cash liquid rather than tying it up in a deposit account. Useful at move-in when cash reserves are low.
- **Legal minimum:** §551 BGB limits the maximum Mietkaution to 3× the monthly net cold rent (Nettokaltmiete). Mietkautionsversicherung must cover the full deposit amount.
- **Status:** Optional alternative to cash deposit. Requires landlord agreement and tenant credit check (Bonitat/Schufa).
- **Coverage type:** Sum-based — the guaranteed deposit amount in EUR.

---

## Coverage Benchmarks

| Criterion | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| Guarantee amount | = full deposit (max 3× Nettokaltmiete) | 3× Nettokaltmiete | Must equal contractually agreed deposit |
| Landlord acceptance | Required | Required | Landlord must agree to accept guarantee in lieu of cash |
| Schufa/Bonitat check | Standard | Standard | Insurer verifies creditworthiness before issuing |
| Annual premium rate | 3–5% of deposit amount | 3–5% | Market range — verify current rates |
| Cost vs. cash deposit | Break-even at ~20–33 years | — | Cash deposit costs opportunity cost only; insurance adds real cost |

> **Cost comparison:**
> ```
> cash_deposit = 3 × nettokaltmiete          # tied-up capital, earns low interest
> annual_premium = deposit_amount × 0.04     # typical 4% of guarantee sum
> break_even_years = deposit_amount / (annual_premium - deposit_interest_opportunity_cost)
> ```
> At 4% annual premium on €2,400 deposit: annual cost = €96. A cash deposit tied at savings rate of 2% foregoes €48/year in opportunity cost. Real annual cost difference: €48/year. After 50 years the insurance costs more.

> **Landlord consent:** Mietkautionsversicherung requires the landlord's explicit agreement to accept a guarantee certificate in lieu of cash. Many landlords prefer cash — verify before purchasing.

---

## Legal Minimums

**§551 BGB (Begrenzung der Mietsicherheit):** The security deposit cannot exceed 3× the monthly net cold rent (Nettokaltmiete). This applies equally to cash deposits and guarantee instruments.

**Alternative instruments:**
- Bankburgschaft (bank guarantee) — bank guarantees deposit to landlord; tenant pays annual fee (~1–2%) or blocks collateral
- Mietkautionsversicherung (insurance guarantee) — insurer guarantees; higher annual premium
- Sparbuch / Depot (savings account/brokerage in landlord's name) — traditional cash deposit

---

## Common Coverage Components

**Mietkautionsversicherung provides:**
- Guarantee certificate (Burgschaftsurkunde) issued to landlord
- Coverage of full deposit amount (up to 3× Nettokaltmiete) for claims by landlord
- Insurer's promise to pay landlord in case of tenant default (unpaid rent, damage, utility arrears)

**What the policy does NOT cover (from tenant's perspective):**
- This product protects the landlord, not the tenant
- If landlord makes a claim and insurer pays, the insurer has full right of recourse (Regress) against the tenant
- Tenant still owes the full deposit amount to the insurer after a claim — they just don't need to pay it upfront

**Insurer's right of recourse:**

> **Critical:** Mietkautionsversicherung is NOT a gift. If the landlord claims against the guarantee and the insurer pays, the insurer will recover the full amount from the tenant via Regresspflicht. This differs from cash deposit where landlord draws from the tenant's own funds.

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Burgschaftsgeber / Versicherungsgesellschaft | string | Full company name |
| premium_monthly | Monatlicher Anteil / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR; typical: deposit × annual rate |
| coverage_amount | Burgschaftssumme / Garantiebetrag | number | EUR; the guaranteed deposit amount |
| start_date | Versicherungsbeginn / Burgschaftsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Leistungsumfang / Garantieleistung | string[] | ["Mietkautionsgarantie", "Burgschaft fur Vermieter"] |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list |

> **coverage_type:** sum_based — `coverage_amount` is the guaranteed deposit sum in EUR. Example: 3 × €800 Nettokaltmiete = €2,400.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Mietkautionsversicherung | Rental deposit insurance/guarantee | Core product |
| Mietkaution | Rental security deposit | What this replaces |
| Nettokaltmiete | Net cold rent | Base rent excluding utilities |
| Kaltmiete | Cold rent | Often used interchangeably |
| Burgschaftsurkunde | Guarantee certificate | Document given to landlord |
| Burgschaft | Guarantee | Legal form of security instrument |
| Bankburgschaft | Bank guarantee | Alternative to insurance guarantee |
| §551 BGB | Security deposit cap (3×) | Maximum deposit legally allowed |
| Regress / Regresspflicht | Right of recourse | Insurer recovers from tenant after paying landlord |
| Bonitat / Schufa | Creditworthiness check | Required before policy issuance |
| Jahresbeitrag | Annual premium | Charged annually for guarantee |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit.

**Policy termination conditions:**
- Tenant moves out and deposit obligation ends — policy should be cancelled at tenancy end
- Landlord returns deposit obligation — policy cancels at landlord's written release
- Tenant switches to cash deposit — insurer issues release letter to landlord

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- Change in coverage terms

**Coordination with tenancy:** The Mietkautionsversicherung term must align with tenancy duration. Do NOT cancel before tenancy ends without landlord's written confirmation that the deposit obligation is released.

---

## Sources

- **§551 BGB (Burgerliches Gesetzbuch):** Statutory cap on Mietkaution (3× Nettokaltmiete)
- **Verbraucherzentrale:** Consumer guidance on Mietkaution alternatives — https://www.verbraucherzentrale.de
- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics — https://www.gdv.de
- **VVG:** Governing insurance contract law
