---
tax_year: 2025
country: germany
domain: kfz-schutzbrief
last_updated: 2026-04-12
source: "GDV, ADAC, Verbraucherzentrale, VVG"
---

# Kfz-Schutzbriefversicherung (Motor Breakdown Assistance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. Coverage geographic scope and roadside assistance response times are service-level specifications that change with insurers.

---

## Overview

Kfz-Schutzbriefversicherung (Schutzbrief) provides roadside breakdown assistance, towing, and mobility services when a vehicle breaks down or is immobilized. It is a service-based product, not a sum-based indemnity policy.

- **Who needs it:** Vehicle owners who do not already have equivalent coverage via ADAC membership or Kfz policy add-on.
- **Legal minimum:** None. Purely optional.
- **Status:** Optional. Often redundant if ADAC membership or Kfz Schutzbrief add-on exists — verify overlap before purchasing standalone policy.
- **Coverage type:** Service-based (not sum-based) — see `coverage_amount` note below.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Pannenhilfe (breakdown assistance) | Germany | EU-wide | Critical: EU-wide coverage is standard for frequent travelers |
| Abschleppen (towing) | To nearest workshop | To workshop of choice | "Nearest workshop" limits repair quality options |
| Mietwagen (rental car) | 1 day | 3–7 days | Covers mobility while vehicle is being repaired |
| Ubernachtungskosten (hotel) | Optional | 1–2 nights | Covers hotel if breakdown prevents return same day |
| Ruckreise (return travel) | Recommended | Included | Train/flight home if vehicle cannot be repaired same day |
| Hotline | 24/7 | 24/7 | Must be available around the clock |
| Geographic scope | Germany | EU + major non-EU destinations | EU-wide is the minimum for travelers |

> **Overlap warning:** ADAC membership (Vollmitgliedschaft) provides equivalent or superior breakdown coverage for all vehicles the member drives, not just owned vehicles. If the user has ADAC membership, a standalone Schutzbrief is likely redundant. Many Kfz-Versicherung policies include Schutzbrief as a paid add-on — check Kfz policy first.

---

## Legal Minimums

No statutory minimum. Schutzbrief is entirely voluntary.

---

## Common Coverage Components

**Typically included (comprehensive Schutzbrief):**
- Pannenhilfe (roadside breakdown assistance — mechanic dispatched)
- Abschleppen zum nachsten Fachbetrieb (towing to nearest qualified workshop)
- Mietwagen (rental car for duration of repair — usually 1–7 days)
- Hotel (overnight accommodation if stranded)
- Ruckreise (travel home if vehicle cannot be repaired same day)
- Fahrzeugruckholung (vehicle retrieval after repair if owner cannot return)
- Krankentransport (transport for injured persons after accident)

**Optional/add-on coverage:**
- Ausland-Pannenhilfe (breakdown abroad — if not included in base)
- Fahrzeugbergung (vehicle recovery from accident scene — may require Vollkasko)
- Tierarztkosten (veterinary costs if animal in vehicle is injured — rare)

**Commonly excluded:**
- Regular maintenance and mechanical failure due to neglect
- Fuel running out (some insurers exclude — check explicitly)
- Tire replacement (some policies only provide roadside assistance, not tire replacement cost)
- Damage covered under Kfz-Haftpflicht or Vollkasko (separate claim)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Versicherer / Schutzbriefanbieter | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | null | null | Service-based — no fixed monetary cap; record null |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Leistungsumfang / Versicherungsschutz | string[] | Pannenhilfe, Abschleppen, Mietwagen, Ruckreise, Hotline |
| exclusions | Ausschlusse / Nicht versichert | string[] | Geographic limits, exclusion list |

> **coverage_type: service_based** — Schutzbrief dispatches services (tow truck, rental car, hotel booking). Do NOT record a fixed monetary sum in `coverage_amount` — record null. Note geographic coverage scope in `notes` field.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Kfz-Schutzbrief | Motor breakdown assistance | Core product |
| Pannenhilfe | Breakdown assistance | Roadside mechanic dispatch |
| Abschleppen | Towing | Vehicle transport to workshop |
| Mietwagen | Rental car | Mobility during repair |
| Ruckreise | Return travel | Getting home while vehicle is repaired |
| Fahrzeugruckholung | Vehicle retrieval | Collecting vehicle after repair |
| Ubernachtungskosten | Hotel costs | Overnight if stranded |
| Hotline | Assistance hotline | 24/7 emergency number |
| ADAC | Auto Club — competitor | Alternative to Schutzbrief policy |
| Inland | Domestic (Germany) | Coverage within Germany |
| Ausland / Europa | Abroad / Europe | EU-wide coverage |
| Selbstbeteiligung | Deductible | Usually none for Schutzbrief |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit (or as specified — may align with Kfz policy renewal).

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After claim settlement — cancel within 4 weeks
- Vehicle deregistration — service product tied to vehicle; policy terminates automatically

**ADAC substitution:** If the user joins ADAC (Vollmitgliedschaft), the standalone Schutzbrief becomes redundant. Cancellation with Sonderkundigungsrecht may not be available for this reason — must wait for next Kundigungsfrist unless premium increased.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics — https://www.gdv.de
- **ADAC (Allgemeiner Deutscher Automobil-Club):** Primary competitor and benchmark — https://www.adac.de
- **Verbraucherzentrale:** Consumer guidance on Kfz-Schutzbrief vs. ADAC membership
- **VVG:** Governing insurance contract law
