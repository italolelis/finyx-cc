---
tax_year: 2025
country: germany
domain: fahrrad
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, VVG, Stiftung Warentest Fahrradversicherung tests"
---

# Fahrradversicherung (Bicycle Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. E-bike coverage requirements are evolving — verify e-bike-specific terms with current policy.

---

## Overview

Fahrradversicherung covers bicycle theft, vandalism, and accident damage. It is available as a standalone policy or as a Zusatz (add-on) to an existing Hausratversicherung. E-bikes have specific coverage requirements distinct from standard bicycles.

- **Who needs it:** Bicycle owners with a bicycle valued above ~€500; especially e-bike owners (high replacement cost).
- **Legal minimum:** None. Purely optional.
- **Status:** Optional. Priority depends on bicycle value and urban theft risk.
- **Coverage type:** Sum-based — insured value of the bicycle in EUR.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Neuwertentschadigung (replacement cost) | Required | Required | Critical: Zeitwert (depreciated value) is insufficient for recent bikes |
| Diebstahl (theft) | Included | Included | Core risk — must require qualified lock |
| Vandalismus | Included | Included | Willful damage |
| Unfallschaden (accident damage) | Included | Included | Collision damage to bicycle |
| E-bike: Akkuschaden (battery damage) | Required for e-bikes | Required | Battery replacement cost is high (€500–2,000) |
| E-bike: Elektronikschaden | Required for e-bikes | Required | Motor and electronics coverage |
| Selbstbeteiligung | €0–150 | €0–50 | Lower deductible = simpler claims for mid-value bikes |
| Schlossanforderung (lock requirement) | Qualify lock required | Approved lock list | Many insurers require a specific quality lock or void coverage |

> **Neuwert vs. Zeitwert:** Neuwertentschadigung pays the current replacement cost of an equivalent bicycle. Zeitwertentschadigung pays the depreciated value (original price minus depreciation per year). For a €2,000 bicycle after 2 years, Zeitwert might pay only €1,000–1,200. Neuwert is the correct benchmark.

> **Hausrat overlap:** Fahrradversicherung as a Hausrat add-on (Fahrrad-Zusatz) typically covers only Diebstahl and caps coverage at €3,000–5,000. Standalone Fahrradversicherung offers broader coverage (accident, vandalism) and higher sums. Verify if the user has both — the standalone may be redundant if the Hausrat add-on is adequate.

---

## Legal Minimums

No statutory minimum. Coverage adequacy is user-driven.

---

## Common Coverage Components

**Typically included (comprehensive standalone policy):**
- Diebstahl (theft after break-in of qualified lock)
- Raub (robbery — theft with force or threat)
- Vandalismus (intentional third-party damage)
- Unfallschaden (collision damage to bicycle)
- Sturmschaden (storm damage)

**E-bike specific (must verify for e-bikes):**
- Akkuschaden (battery damage — defect, fire, accident)
- Elektronik- und Motorschaden (motor and electronics failure or damage)
- Ladekabelschaden (charger cable damage)

**Optional/add-on:**
- Pannenhilfe (breakdown assistance — rare in pure bicycle policies)
- Zubehur (accessories — helmet, lights, panniers)
- Verschleiss (wear and tear — rarely included; check explicitly)

**Commonly excluded:**
- Diebstahl ohne qualifiziertes Schloss (theft without approved lock)
- Verschleiss (normal wear and tear)
- Vorsatzliche Schaden (intentional damage by insured)
- Schaden durch Dritte mit Haftpflicht (damage by identified third parties with liability — their Haftpflicht pays)
- Racing/competition use (unless specifically included)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Fahrradversicherer / Versicherungsgesellschaft | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | Versicherungssumme / Neuwert | number | EUR; replacement value of bicycle |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Versicherungsschutz / Leistungsumfang | string[] | Diebstahl, Vandalismus, Unfall, Akkuschaden, Elektronik |
| exclusions | Ausschlusse / Nicht versichert | string[] | Lock requirements, exclusion list |

> **coverage_type:** sum_based — the `coverage_amount` is the insured replacement value of the bicycle (Neuwert). Record the insured sum in EUR.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Fahrradversicherung | Bicycle insurance | Core product |
| Neuwert | Replacement cost | Current value of equivalent new bike |
| Zeitwert | Depreciated value | Inferior — avoid policies with Zeitwert only |
| Neuwertentschadigung | Replacement cost payout | Key benchmark |
| Diebstahl | Theft | Must require qualified lock |
| Qualifiziertes Schloss | Qualified lock | Approved lock — required for theft claim |
| Vandalismus | Vandalism | Malicious third-party damage |
| Unfallschaden | Accident damage | Collision damage to bicycle |
| E-Bike / Pedelec | Electric bicycle | ≤25 km/h motor assist |
| S-Pedelec | Speed pedelec | >25 km/h — may require Kfz registration/insurance |
| Akkuschaden | Battery damage | E-bike specific |
| Elektronikschaden | Electronics damage | E-bike motor and components |
| Fahrrad-Zusatz | Bicycle add-on | Hausrat policy rider |
| Selbstbeteiligung | Deductible | Per-claim excess |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit.

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After claim settlement — cancel within 4 weeks
- Bicycle sold or stolen (covered loss) — policy may be cancelled

**Note on e-bike changes:** If the insured bicycle changes (e.g., upgrades from standard to e-bike), the insurer must be notified. Coverage may require adjustment. Failure to notify voids coverage.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics — https://www.gdv.de
- **Verbraucherzentrale:** Consumer guidance on Fahrradversicherung — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Comparative test results for bicycle insurance
- **VVG:** Governing insurance contract law
