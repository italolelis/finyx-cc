---
tax_year: 2025
country: germany
domain: hausrat
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, VVG, Stiftung Warentest Hausratversicherung tests"
---

# Hausratversicherung (Household Contents Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. Coverage thresholds are market-driven — per-sqm values should be verified annually against GDV statistics.

---

## Overview

Hausratversicherung covers the household contents (Hausrat) against damage or loss from fire, water, burglary, storm, and other covered perils. It protects moveable property within the insured dwelling — furniture, electronics, clothing, valuables, and appliances.

- **Who needs it:** All homeowners and renters with significant household contents.
- **Legal minimum:** None. Purely optional.
- **Status:** Optional but strongly recommended. Underinsurance is the most common failure mode.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Versicherungssumme | ≥€650/m² × Wohnflache | ≥€650/m² × Wohnflache with Unterversicherungsverzicht | Key benchmark from GDV/Verbraucherzentrale |
| Unterversicherungsverzicht | Required | Required | Insurer waives Unterversicherungseinwand if sum ≥ threshold |
| Einbruchdiebstahl (Einbruch) | Included | Included | Theft after forced entry |
| Leitungswasser | Included | Included | Leaking pipes/water damage |
| Elementarschaden | Optional | Recommended | Flooding, landslides — increasingly critical |
| Fahrrad-Zusatz | Optional | Up to €3,000–5,000 | Bicycle theft rider — check overlap with standalone Fahrradversicherung |

> **Underinsurance warning:** If the Versicherungssumme is below the replacement value of household contents, the insurer pays proportionally less. The Unterversicherungsverzicht clause eliminates this pro-ration when the insured sum meets the threshold.

**Versicherungssumme calculation:**

```
min_coverage = living_area_sqm × 650
recommended_coverage = max(min_coverage, replacement_value_of_all_contents)
```

Example: 75m² apartment → minimum Versicherungssumme = 75 × 650 = €48,750

---

## Legal Minimums

No statutory minimum. Coverage adequacy is the user's responsibility. The €650/m² benchmark is a GDV/Verbraucherzentrale recommendation, not a legal mandate.

---

## Common Coverage Components

**Typically included (good policy):**
- Feuer (fire, lightning, explosion, implosion)
- Einbruchdiebstahl (burglary and break-in theft)
- Raub (robbery)
- Leitungswasser (burst/leaking pipes, washing machine overflow)
- Sturm ab Windstarke 8 (storm from wind force 8)
- Hagel (hail)
- Glasbruch (glass breakage — sometimes separate tariff)
- Kurzschlussschaden an Elektrogeratenn (electrical short-circuit damage)

**Optional/add-on coverage:**
- Elementarschaden (Uberschwemmung, Ruickstau, Erdbeben, Erdrutsch, Schneedruck)
- Fahrrad-Zusatz (bicycle theft up to defined sum)
- Wertsachen (cash, jewelry — limited sub-limits apply)
- Aquarienschaden (aquarium water damage)
- Haustierbedingte Schaden (pet damage — rare add-on)

**Commonly excluded:**
- Fahrlassigkeit (some policies exclude gross negligence — check VVG §81)
- Wertsachen above sub-limits (cash, jewelry, watches — verify sub-limits)
- Schaden durch Krieg/Kernenergie (war, nuclear — excluded by VVG)
- Schaden durch Vorsatz (intentional damage)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Versicherer / Versicherungsunternehmen | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | Versicherungssumme | number | EUR total; e.g., 48750 |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Leistungsumfang / Versicherte Gefahren | string[] | Perils covered: Feuer, Einbruch, etc. |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list from policy |

> **coverage_type:** sum_based — the Versicherungssumme is a monetary cap on total claim payout.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Hausratversicherung | Household contents insurance | Core product |
| Versicherungssumme | Sum insured | Total coverage amount |
| Unterversicherungsverzicht | Underinsurance waiver | Insurer waives pro-ration if sum meets threshold |
| Einbruchdiebstahl | Burglary theft | Theft after forced entry |
| Leitungswasser | Water pipe damage | Internal water damage |
| Sturm | Storm damage | Wind force ≥ 8 |
| Elementarschaden | Natural disaster damage | Flood, landslide, earthquake |
| Fahrrad-Zusatz | Bicycle rider | Add-on for bicycle theft |
| Wohnflache | Living area | Used to calculate minimum sum |
| Wertsachen | Valuables | Cash, jewelry — sub-limits apply |
| Selbstbeteiligung | Deductible | Amount insured pays per claim |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit (policy anniversary).

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After a claim is paid — cancel within 4 weeks of settlement
- Change in risk (e.g., move to new address) — must notify insurer; insurer may adjust or offer new terms

**Address change (Umzug):** Policy must be transferred to new address. Temporary coverage at both addresses during move. New address may affect risk class and premium. Insured has Sonderkundigungsrecht if premium increases at new address.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Benchmark data and industry statistics — https://www.gdv.de
- **Verbraucherzentrale:** Consumer guidance on Hausratversicherung, underinsurance calculator — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Comparative test results for Hausrat policies
- **VVG §81:** Gross negligence and insurer's right to reduce claim payments
