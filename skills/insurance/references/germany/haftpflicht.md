---
tax_year: 2025
country: germany
domain: haftpflicht
last_updated: 2026-04-12
source: "GDV, Verbraucherzentrale, §823 BGB (Deliktshaftung), §1 PflVG, VVG (Versicherungsvertragsgesetz)"
---

# Privathaftpflichtversicherung (Personal Liability Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. Coverage minimums are not legislated for private liability — benchmarks are market-driven consensus from GDV and Verbraucherzentrale.

---

## Overview

Privathaftpflichtversicherung protects insured persons against financial claims from third parties arising from negligent damage to persons, property, or financial interests (§823 BGB). It is one of the most important personal insurance products in Germany — without it, a single accident causing serious injury can result in lifetime wage garnishment.

- **Who needs it:** Every adult resident in Germany. Strongly recommended.
- **Legal minimum:** None for private individuals. Mandatory for specific professions and vehicle owners (Kfz).
- **Status:** Optional but universally recommended.

---

## Coverage Benchmarks

| Criterion | Minimum Acceptable | Recommended | Notes |
|-----------|-------------------|-------------|-------|
| Deckungssumme (pauschal) | €5,000,000 | €10,000,000–50,000,000 | Pauschal covers Personen-, Sach-, and Vermogensschaden combined |
| Mietsachschaden | Included | Included | Damage to rented property — commonly excluded, must verify |
| Schlusselverlust | Included | Included | Loss of entrusted keys (not own keys) |
| Gefalligkeitsschaden | Included | Included | Damage while performing informal favors |
| Forderungsausfalldeckung | Included | Included | Covers insured when at-fault third party is uninsured |
| Selbstbeteiligung | €0–150 | €0 | Lower deductibles reduce claim friction for minor incidents |

> **Critical:** Deckungssumme ≥ €5,000,000 is the minimum the research agent should flag. Policies below this threshold are structurally under-insured given German personal injury liability precedents.

---

## Legal Minimums

No statutory minimum for private Haftpflicht. The §823 BGB liability obligation is unlimited — the benchmark is driven by what is adequate to cover realistic worst-case personal injury claims.

---

## Common Coverage Components

**Typically included (good policy):**
- Personenschaden (bodily injury to third parties)
- Sachschaden (property damage to third parties)
- Vermogensschaden (financial loss to third parties)
- Mietsachschaden (damage to rented apartment/house)
- Schlusselverlust (loss of entrusted keys)
- Gefalligkeitsschaden (damage while helping someone)
- Forderungsausfalldeckung (insured's recovery when third party is uninsured)
- Deliktunfahige Personen (children under 7 and mentally incapacitated individuals)

**Often excluded or limited (verify):**
- Vorsatzliche Schaden (intentional damage — excluded by law, VVG)
- Eigene Sachschaden (own property damage — not covered)
- Berufliche Tatigkeit (professional activities — requires separate Berufshaftpflicht)
- Kraftfahrzeuge (vehicles — requires separate Kfz-Haftpflicht)
- Schaden durch Haustiere with high-risk breeds (may require Hundehaftpflicht)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | Versicherer / Versicherungsunternehmen | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahresperson | number | EUR |
| coverage_amount | Deckungssumme (pauschal) | number | EUR; e.g., 5000000 |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit / Beitragsfalligkeitsdatum | ISO date | Annual renewal date |
| kuendigungsfrist_months | Kundigungsfrist | number | Convert "3 Monate" → 3 |
| coverage_components | Leistungsumfang / Versicherungsschutz | string[] | Line items from coverage table |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list from policy |

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Privathaftpflicht | Personal liability | Core coverage type |
| Deckungssumme | Coverage limit | Total sum insured |
| Pauschal | Combined/flat | Single sum covering all damage types |
| Personenschaden | Bodily injury | Injury to third parties |
| Sachschaden | Property damage | Damage to third-party property |
| Vermogensschaden | Financial loss | Economic loss to third parties |
| Mietsachschaden | Rental property damage | Damage to rented home |
| Schlusselverlust | Key loss | Loss of entrusted keys |
| Gefalligkeitsschaden | Favor damage | Damage while helping informally |
| Forderungsausfall | Uninsured third-party | Coverage when at-fault party is uninsured |
| Selbstbeteiligung | Deductible | Amount insured pays per claim |
| Versicherungsbeginn | Policy start date | Inception date |
| Hauptfalligkeit | Renewal date | Annual policy renewal |
| Kundigungsfrist | Notice period | Cancellation lead time |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 3 months before policy anniversary (Hauptfalligkeit).

**Sonderkundigungsrecht triggers:**
- Premium increase (Beitragserhohung) — insured may cancel within 4 weeks of notification
- Claim settlement where insurer pays — insured may cancel within 4 weeks of settlement letter
- Policy anniversary (if Kundigungsfrist met)
- Change in risk profile (e.g., household members change) — insurers may adjust or terminate

**Tacit renewal (stillschweigende Verlangerung):** Most policies auto-renew for 1 year if not cancelled within the Kundigungsfrist. Deadline tracking is critical.

---

## Sources

- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** Industry statistics and benchmark data — https://www.gdv.de
- **Verbraucherzentrale:** Consumer guidance on Haftpflichtversicherung — https://www.verbraucherzentrale.de
- **Stiftung Warentest / test.de:** Periodic comparative tests of Haftpflicht policies
- **§823 BGB:** Legal basis for tort liability in Germany
- **VVG (Versicherungsvertragsgesetz):** Insurance contract law governing all private insurance policies
