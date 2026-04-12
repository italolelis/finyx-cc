---
tax_year: 2025
country: germany
domain: risikoleben
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, §10 EStG, VVG, Stiftung Warentest Risikolebensversicherung tests"
---

# Risikolebensversicherung (Term Life Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. Premium tables and health underwriting requirements are insurer-specific and should be verified via live research. §10 EStG deductibility limits are set annually.

---

## Overview

Risikolebensversicherung pays a defined lump sum (Versicherungssumme) to the named beneficiary if the insured dies within the policy term. It provides pure death protection — no savings component, no cash value. It is the most cost-effective way to protect financial dependents.

- **Who needs it:** Any person with financial dependents (partner, children, co-borrower on a mortgage).
- **Legal minimum:** None. Purely optional.
- **Status:** Optional but strongly recommended when dependents exist or a mortgage co-obligor is present.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Versicherungssumme (sum insured) | 3× annual gross income | 3–5× annual gross income | If dependents exist |
| Policy term | Until youngest child is 25 | Until retirement age or mortgage payoff | Covers dependency period |
| Coverage structure | Level term | Level or declining (Annuitat) | See term types below |
| Nachversicherungsgarantie | Recommended | Recommended | Option to increase sum without new health check |
| Beitragsbefreiung bei BU | Optional | Recommended | Premiums waived if insured becomes occupationally disabled |

> **Benchmark calculation:**
> ```
> recommended_sum = max(
>   annual_gross_income × 5,
>   outstanding_mortgage_balance
> )
> ```
> If a mortgage exists, the Versicherungssumme should at minimum cover the outstanding debt balance so the surviving partner can clear the loan.

---

## Term Types

| Type | German Term | Structure | Best For |
|------|-------------|-----------|---------|
| Level term | Konstante Versicherungssumme | Fixed sum throughout term | Income replacement, single lump-sum need |
| Declining term (linear) | Linear fallende Versicherungssumme | Sum decreases equally each year | Mortgage payoff (matches amortization) |
| Declining term (annuity) | Annuitat fallende Versicherungssumme | Sum decreases at loan amortization rate | Matched to annuity mortgage schedule |

> **For mortgage protection:** Declining term (Annuitat) is usually most cost-effective — the Versicherungssumme tracks the outstanding balance. A level term policy over-insures in the later years and costs more in premiums.

---

## Legal Minimums

No statutory minimum. Recommended sums are financial planning benchmarks, not legally mandated.

---

## Health Underwriting

Unlike most other insurance types, Risikoleben requires health underwriting — similar in rigor to PKV (Private Krankenversicherung). The insurer assesses the risk of death during the policy term.

**Health questionnaire types:**
- Simplified underwriting: short questionnaire for lower sums (typically <€250,000) or younger applicants
- Full underwriting: medical questionnaire, possible medical exam or doctor report for higher sums

**Impact on premium:**
- Clean health history: standard rate
- Controlled chronic conditions: Risikozuschlag (surcharge, typically 25–100%)
- High-risk conditions: possible exclusion of specific causes of death, or outright rejection

> **Key risk:** Misrepresentation on health questions (arglistige Tauschung or grobe Fahrlassigkeit) voids the policy. Beneficiaries may not receive the sum if undisclosed conditions are discovered after death. Answer all health questions truthfully and completely.

---

## Common Coverage Components

**Standard coverage:**
- Todesfallschutz (death benefit paid to beneficiary)
- Coverage for all causes of death (unless explicitly excluded)

**Optional riders (Zusatzbausteine):**
- Nachversicherungsgarantie (option to increase sum without health re-underwriting — triggered by marriage, birth, property purchase)
- Beitragsbefreiung bei Berufsunfahigkeit (BU) — premiums waived if insured is occupationally disabled
- Unfalltod-Zusatz (double indemnity on accidental death — limited additional value)

**Commonly excluded:**
- Suicide within first 3 years (suicide exclusion — Selbstmord within Karenzzeit)
- Death from risky activities (may require disclosure: extreme sports, aviation as pilot)
- Death from intentional illegal acts by the beneficiary

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Lebensversicherer / Versicherungsgesellschaft | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | Versicherungssumme | number | EUR; e.g., 250000 for level term; record initial sum for declining term |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Vertragsende / Ablaufdatum | ISO date | End of policy term (NOT annual renewal — Risikoleben is fixed-term) |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3; note: fixed-term policies have limited cancellation rights |
| coverage_components | Versicherungsumfang / Leistungen | string[] | Todesfallschutz, Nachversicherungsgarantie, Beitragsbefreiung-BU |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list (suicide clause, specific activities) |

> **renewal_date for Risikoleben:** Record as the policy end date (Vertragsende / Ablauf), not an annual renewal. Risikoleben is a fixed-term contract; it does not auto-renew — it expires.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Risikolebensversicherung | Term life insurance | Pure death protection |
| Versicherungssumme | Sum insured / death benefit | Lump sum paid on death |
| Todesfallschutz | Death benefit | Core coverage |
| Versicherungsnehmer | Policyholder | May differ from Versicherte Person |
| Versicherte Person | Insured person | Person whose life is covered |
| Bezugsberechtigte | Beneficiary | Person who receives death benefit |
| Nachversicherungsgarantie | Option to increase sum | No re-underwriting at life events |
| Beitragsbefreiung | Premium waiver | Premiums waived on disability |
| Berufsunfahigkeit (BU) | Occupational disability | Triggers premium waiver add-on |
| Risikozuschlag | Risk surcharge | Extra premium for health risk |
| Konstante Versicherungssumme | Level term | Fixed sum throughout |
| Fallende Versicherungssumme | Declining term | Decreasing sum — for mortgage |
| Vertragsende / Ablauf | Policy end date | Fixed term expiry — not annual renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Fixed-term contract:** Risikoleben does not auto-renew. At Vertragsende, coverage ceases. A new policy requires new health underwriting at current age — premiums will be higher.

**Mid-term cancellation:**
- Kundigungsfrist: typically 3 months
- No cash value — cancellation forfeits all paid premiums with no refund
- Early cancellation may be appropriate if the insured risk (mortgage, dependents) has materially changed

**Sonderkundigungsrecht triggers:**
- Premium increase (Beitragserhohung) — cancel within 4 weeks
- Change in coverage terms — cancel within 4 weeks

**Note:** Unlike savings insurance (Kapitallebensversicherung), Risikoleben has no Ruckkaufswert (surrender value). All premiums are consumed by pure risk premium.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics — https://www.gdv.de
- **Verbraucherzentrale:** Consumer guidance on Risikolebensversicherung — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Comparative test results for Risikoleben policies
- **VVG:** Governing insurance contract law (health disclosure obligations, suicide clause rules)
- **§10 EStG:** Limited deductibility context (Risikoleben premiums are generally not deductible unless part of Basisversorgung — which they are not)
