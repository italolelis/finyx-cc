---
tax_year: 2025
country: germany
domain: zahnzusatz
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, §55 SGB V (GKV Festzuschuss), Stiftung Warentest Zahnzusatz tests"
---

# Zahnzusatzversicherung (Supplemental Dental Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. GKV Festzuschuss rates are set annually by the G-BA (Gemeinsamer Bundesausschuss) — verify before acting. The bonus booklet (Bonusheft) requirement is unchanged.

---

## Overview

Zahnzusatzversicherung supplements the statutory GKV dental coverage. GKV pays only a base Festzuschuss (fixed grant) for dental prosthetics — typically 60% of the reference value (Regelversorgung). The actual patient cost for quality dental work is significantly higher. Zahnzusatz bridges this gap.

- **Who needs it:** GKV members who want comprehensive dental coverage. Particularly valuable for those needing prosthetics, implants, or orthodontic work.
- **Legal minimum:** None. Purely optional supplement to GKV.
- **Status:** Optional but high-value for GKV members. Priority increases with age and dental history.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Zahnersatz (prosthetics) reimbursement | ≥70% of total cost | ≥80–100% | Including implants where possible |
| Zahnbehandlung (treatment) reimbursement | ≥70% of cost above GKV | ≥80–100% | Root canals, complex fillings |
| Professionelle Zahnreinigung (PZR) | 1–2x annually included | 2x annually | GKV covers PZR only in limited cases |
| Wartezeit | ≤3 months ideally | No Wartezeit | Immediate coverage is better |
| Staffelung | Max EUR 500 in year 1 acceptable | No Staffelung ideally | Graduated annual cap reduces value significantly |
| Implantate | Covered with good policy | Included | Many policies exclude or cap implants |
| Kieferorthopadie (KFO) | Relevant for families | Included for children | Adult orthodontics often excluded |

> **GKV baseline context:** GKV pays Festzuschuss based on Regelversorgung (reference treatment). For 2025 with a complete Bonusheft (5-year bonus chain): **70% of Festzuschuss**. The typical GKV-covered share of actual prosthetic costs is 50–60% of the bill — the patient pays the gap. Zahnzusatz should cover this gap.

**Staffelung (graduated coverage) — common pattern:**

| Policy Year | Annual Zahnersatz Cap (example) |
|-------------|--------------------------------|
| Year 1 | €500–1,000 |
| Year 2 | €1,000–2,000 |
| Year 3 | €2,000–3,500 |
| Year 4+ | €3,500–unlimited |

> Staffelung makes the first 2 years of coverage limited. Users with immediate dental needs should seek no-Staffelung policies or accept that initial claims are capped.

---

## Legal Minimums

No statutory minimum. The GKV Festzuschuss system sets the floor — §55 SGB V defines the Festzuschuss rates and bonus increments.

**GKV bonus system (Bonusheft):**

| Bonusheft status | GKV Festzuschuss |
|-----------------|-----------------|
| No Bonusheft | 60% of Regelversorgung |
| 5-year uninterrupted check-up record | 70% of Regelversorgung |
| 10-year uninterrupted check-up record | 75% of Regelversorgung |

---

## Common Coverage Components

**Typically included (comprehensive policy):**
- Zahnersatz (crowns, bridges, dentures — partial and full)
- Zahnbehandlung (root canals, complex fillings beyond GKV standard)
- Professionelle Zahnreinigung (PZR — professional cleaning)
- Inlays (ceramic/gold inlays beyond Regelversorgung)

**Optional/add-on:**
- Implantate (dental implants — coverage varies widely; many policies cap or exclude)
- Kieferorthopadie (orthodontics — often only for children under 18)
- Bleaching / kosmetische Behandlungen (cosmetic — usually excluded)
- Funktionstherapie (jaw dysfunction therapy — some policies include)

**Commonly excluded:**
- Vorerkrankungen (pre-existing conditions at policy start — common exclusion or Wartezeit basis)
- Behandlungen vor Vertragsbeginn (treatment begun before policy start)
- Bleaching and cosmetic procedures
- Adult orthodontics in many policies

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Versicherer / Zahnversicherer | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | Jahreslimit Zahnersatz (year 1) | number/null | Record first-year Staffelung cap; null if unlimited |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Versicherungsschutz / Leistungsumfang | string[] | Zahnersatz, Zahnbehandlung, PZR, Implantate, KFO |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list |

> **coverage_type:** sum_based (Staffelung-based annual cap). Record the year-1 cap in `coverage_amount` if Staffelung applies; record null only if truly unlimited.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Zahnzusatzversicherung | Supplemental dental insurance | Core product |
| Zahnersatz | Dental prosthetics | Crowns, bridges, dentures |
| Zahnbehandlung | Dental treatment | Non-prosthetic treatment |
| Professionelle Zahnreinigung (PZR) | Professional dental cleaning | Often 2x/year |
| Implantate | Dental implants | Often separate or capped |
| Kieferorthopadie (KFO) | Orthodontics | Braces etc. |
| Festzuschuss | Fixed subsidy | GKV base contribution |
| Regelversorgung | Standard treatment | GKV reference treatment level |
| Staffelung | Graduated coverage | Annual cap increasing over years |
| Wartezeit | Waiting period | Standard: 3 months |
| Bonusheft | Dental check-up record | Required for GKV bonus |
| Eigenanteil | Patient co-payment | Gap between GKV + Zahnzusatz and actual cost |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before Hauptfalligkeit.

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After claim settlement — cancel within 4 weeks
- Change in coverage terms (Bedingungsanderung)

**Switching note:** Switching Zahnzusatz policies typically means a new Wartezeit. Mid-treatment switching should be avoided — treatments begun under the old policy may not be covered by the new policy.

---

## Sources

- **§55 SGB V:** GKV Festzuschuss calculation and bonus system
- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry data — https://www.gdv.de
- **Verbraucherzentrale:** Consumer guidance on Zahnzusatz — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Comparative test results for Zahnzusatz policies (published regularly)
- **G-BA (Gemeinsamer Bundesausschuss):** Regelversorgung definitions and Festzuschuss rates — https://www.g-ba.de
