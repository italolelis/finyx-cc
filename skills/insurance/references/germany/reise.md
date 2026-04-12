---
tax_year: 2025
country: germany
domain: reise
last_updated: 2026-04-12
source: "GDV, BaFin, Verbraucherzentrale, Stiftung Warentest Reiseversicherung tests"
---

# Reiseversicherung (Travel Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. Coverage caps and trip duration limits are product-specific — verify against current policy terms.

---

## Overview

Reiseversicherung bundles several travel-related risks into one or multiple policies. Core components are Auslandskrankenversicherung (foreign health coverage), Reiserucktrittsversicherung (trip cancellation), Reisegepuckversicherung (luggage), and Reisehaftpflicht (travel liability).

- **Who needs it:** Anyone traveling internationally. GKV provides only emergency EHIC coverage within the EU — it does NOT cover medical repatriation (Krankenrucktransport), which can cost €50,000–€300,000.
- **Legal minimum:** None. Purely optional.
- **Status:** Optional but strongly recommended for international travel.
- **Coverage type:** Service-based (not sum-based) — see `coverage_amount` note below.

---

## Coverage Benchmarks

| Component | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Auslandskrankenversicherung | Included | Included — unlimited medical | Core requirement |
| Krankenrucktransport (medical repatriation) | Included | Included — unlimited | Can cost €50,000–€300,000 |
| Reiserucktrittsversicherung | 80% of trip cost | 100% of trip cost | Trip cancellation due to illness, death, disaster |
| Reisegepuckversicherung | Optional | Optional | Luggage loss — check credit card coverage first |
| Reisehaftpflicht | Recommended | Included | Damage caused abroad — may overlap with Haftpflicht policy |
| Trip duration per-trip limit | ≥42 days | ≥56 days or unlimited | KEY: many policies cap individual trip duration at 42 days |
| Selbstbeteiligung | €0–100 | €0 | Lower deductible = simpler claims |

> **CRITICAL: Per-trip duration cap.** Many annual travel policies cap each individual trip at 42 or 56 days. If a single trip exceeds this limit, coverage lapses for the remainder of that trip. Frequent or long-haul travelers must verify this limit explicitly.

> **GKV gap warning:** The European Health Insurance Card (EHIC) provides emergency GKV coverage within the EU/EEA, but does NOT cover: medical repatriation, ambulance transfers across borders, non-emergency care, or care outside the EU. Even for EU travel, Auslandskrankenversicherung is recommended.

---

## Legal Minimums

No statutory minimum. Travel insurance is entirely voluntary.

---

## Common Coverage Components

**Typically included in comprehensive bundle:**
- Auslandskrankenversicherung (foreign health coverage — unlimited medical costs)
- Krankenrucktransport (medically necessary repatriation to Germany)
- Notfalltransport (emergency evacuation)
- Reiserucktritt (trip cancellation — illness, death of close relative, job loss)
- Reiseabbruch (trip interruption — early return)
- Reisegepuck (luggage loss, damage, delay — often capped at €1,000–3,000)
- Reisehaftpflicht (liability for damage caused abroad)

**Optional/add-on modules:**
- Skifahren-Unfall (ski accident coverage)
- Extremsportarten (adventure sports rider)
- COVID-19 (pandemic cancellation — check explicitly; most policies now include)
- Mietwagen-Vollkasko (rental car collision coverage)

**Commonly excluded:**
- Pre-existing medical conditions (Vorerkrankungen) — often excluded or require disclosure
- War zones / political unrest destinations
- Professional sports injuries
- Trips longer than policy duration limit (per-trip cap)
- Non-cancellation reasons not specified in contract (e.g., personal preference)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Reiseversicherer / Versicherungsgesellschaft | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | null | null | Service-based — no fixed monetary cap; record null |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Versicherungsschutz / Leistungsbausteine | string[] | Auslandskranken, Rucktritt, Gepack, Haftpflicht, Rucktransport |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list; note per-trip duration limit |

> **coverage_type: service_based** — Reiseversicherung pays actual costs on a per-event basis. Do NOT record a fixed monetary sum in `coverage_amount` — record null. Note per-trip duration cap in `notes` field.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Reiseversicherung | Travel insurance | General term |
| Auslandskrankenversicherung | Foreign health insurance | Core component |
| Krankenrucktransport | Medical repatriation | Evacuation to Germany |
| Reiserucktritt | Trip cancellation | Pre-departure cancellation |
| Reiseabbruch | Trip interruption | Early return from trip |
| Reisegepuck | Travel luggage | Baggage loss/damage |
| Reisehaftpflicht | Travel liability | Damage caused abroad |
| Selbstbeteiligung | Deductible | Per-claim excess |
| Vorerkrankung | Pre-existing condition | Often excluded |
| Reisedauer | Trip duration | Per-trip day limit — critical |
| Jahreskarte | Annual policy | Covers multiple trips per year |
| Einmalreise | Single trip policy | Covers one specific trip |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit (for annual policies).

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After claim settlement — cancel within 4 weeks
- Change in coverage terms

**Single-trip policies:** Non-cancellable once issued; terminate automatically at end of trip.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics — https://www.gdv.de
- **BaFin (Bundesanstalt fur Finanzdienstleistungsaufsicht):** Consumer information on travel insurance — https://www.bafin.de
- **Verbraucherzentrale:** Consumer guidance on Reiseversicherung — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Comparative test results for travel insurance (published regularly)
