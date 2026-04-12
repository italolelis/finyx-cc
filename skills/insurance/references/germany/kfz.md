---
tax_year: 2025
country: germany
domain: kfz
last_updated: 2026-04-12
source: "§1 PflVG (Pflichtversicherungsgesetz), GDV, Kraftfahrtbundesamt (KBA), VVG, ADAC"
---

# Kfz-Versicherung (Motor Vehicle Insurance) Reference

> **Tax year notice:** This document reflects benchmarks for 2025. SF-Klasse tables and Typklasse/Regionalklasse classifications are published annually by GDV and the Kraftfahrtbundesamt.

---

## Overview

Kfz-Versicherung is the only insurance type in this reference set that is **legally mandatory** in Germany. Every vehicle registered and driven on public roads must have at minimum Kfz-Haftpflicht (third-party liability). Additional coverage tiers (Teilkasko, Vollkasko) are optional but strongly recommended for financed or high-value vehicles.

- **Who needs it:** Every vehicle owner/operator in Germany.
- **Legal minimum:** Kfz-Haftpflicht is mandatory under §1 PflVG. Driving without it is a criminal offense.
- **Status:** Mandatory (Haftpflicht); Optional (Teilkasko, Vollkasko).

---

## Coverage Tiers

**One policy entry per Kfz contract** — NOT one entry per coverage level. Use `coverage_components` to capture which tiers are active (e.g., `["Haftpflicht", "Teilkasko"]`).

### Tier 1: Kfz-Haftpflicht (mandatory)

Covers third-party damage caused by the insured vehicle. Required by §1 PflVG.

| Criterion | Statutory Minimum | Market Standard |
|-----------|-------------------|----------------|
| Personenschaden | €7,500,000 | €100,000,000+ |
| Sachschaden | €1,220,000 | €100,000,000+ |
| Vermogensschaden | €50,000 | €15,000,000+ |

> The statutory minimum is set by §4 PflVG. Most quality insurers offer much higher limits. The legal minimum is **insufficient** — policies with statutory minimums only are a risk.

### Tier 2: Teilkasko (partial comprehensive)

Covers own-vehicle damage from specific named perils. Optional but standard.

| Covered Peril | Notes |
|---------------|-------|
| Feuer und Explosion | Vehicle fire |
| Diebstahl | Theft of entire vehicle |
| Glasbruch | Windscreen/window breakage |
| Kurzschluss | Electrical short-circuit |
| Sturm, Hagel, Blitzschlag, Uberschwemmung | Weather events |
| Wildunfall | Collision with animals (Tier im Sinne des BJagdG) |
| Marderbiss | Marten bite damage to wiring |

**Not covered by Teilkasko:** Self-caused accidents, vandalism, collision damage.

### Tier 3: Vollkasko (fully comprehensive)

Includes all Teilkasko coverage PLUS:

| Additional Coverage | Notes |
|---------------------|-------|
| Selbstverschuldete Unfallschaden | Own-fault accident damage |
| Vandalismusschaden | Malicious third-party damage |
| Parkschaden durch unbekannte Dritte | Hit-and-run damage in parking |
| Fahrzeugteile die gestohlen werden | Parts theft |

> **Benchmark:** Vollkasko is strongly recommended for financed vehicles (Finanzierungsauflagen often require it) and vehicles under 3 years old. For vehicles >7 years old, Teilkasko may be more cost-effective.

---

## Coverage Benchmarks

| Vehicle Age | Recommended Tier | Rationale |
|-------------|-----------------|-----------|
| 0–3 years | Vollkasko | High replacement cost; often required by financing |
| 3–7 years | Vollkasko or Teilkasko | Balance of premium vs. vehicle value |
| >7 years | Teilkasko or Haftpflicht only | Vehicle residual value may not justify Vollkasko premium |
| Financed vehicle (any age) | Vollkasko | Financing contract typically requires it |

---

## Legal Minimums

**§1 PflVG (Pflichtversicherungsgesetz):** Kfz-Haftpflicht is mandatory. Minimum limits set by §4 PflVG (see Coverage Tiers above). Operating an uninsured vehicle on public roads is a criminal offense (§6 PflVG) and results in automatic deregistration by Zulassungsbehorde.

---

## Rating Factors

Understanding these factors is critical for market comparison:

| Factor | Description | Source |
|--------|-------------|--------|
| SF-Klasse (Schadenfreiheitsklasse) | No-claims discount class; starts at SF 0 for new drivers, improves with claim-free years | GDV annual table |
| Typklasse | Risk class of vehicle model based on historical claims data | Kraftfahrtbundesamt (KBA) annual publication |
| Regionalklasse | Risk class based on registration district (Zulassungsbezirk) | GDV annual publication |
| Selbstbeteiligung | Deductible — common options: €150 (Teilkasko), €300–500 (Vollkasko) | Policy-specific |

**SF-Klasse impact:** SF 35 (35 claim-free years) can reduce Haftpflicht premium by 70–80% vs. SF 0 base rate. One at-fault accident typically results in regression by 4–6 SF classes.

---

## Common Coverage Components

**coverage_components field values (string array):**
- `"Haftpflicht"` — mandatory third-party liability
- `"Teilkasko"` — partial comprehensive (named perils)
- `"Vollkasko"` — full comprehensive (includes own fault)
- `"Schutzbrief"` — breakdown and roadside assistance rider (may overlap with Kfz-Schutzbrief standalone policy)
- `"Fahrerschutz"` — driver injury protection (covers own driver — not included in Haftpflicht)

---

## Field Extraction Schema

The document reader agent extracts these fields from policy PDFs. Field names match `insurance.policies[]` in `.finyx/profile.json`.

| Field | German Label in Policy | Type | Notes |
|-------|----------------------|------|-------|
| provider | KFZ-Versicherer / Versicherungsgesellschaft | string | Full company name |
| premium_monthly | Monatsbeitrag / Monatspramie | number | EUR; convert from annual if needed |
| premium_annual | Jahresbeitrag / Jahrespramie | number | EUR |
| coverage_amount | Deckungssumme Haftpflicht | number | EUR for Haftpflicht tier; null for Teilkasko/Vollkasko (service-based repair) |
| start_date | Versicherungsbeginn | ISO date | Format: YYYY-MM-DD |
| renewal_date | Hauptfalligkeit | ISO date | Annual renewal — always 01.01 for most Kfz |
| kuendigungsfrist_months | Kundigungsfrist | number | Standard: 1 month before 31.12 (= Kundigungsfrist 1 Monat) |
| coverage_components | Versicherungsschutz / Deckungsart | string[] | ["Haftpflicht"], ["Haftpflicht","Teilkasko"], or ["Haftpflicht","Teilkasko","Vollkasko"] |
| exclusions | Ausschlusse / Nicht versichert | string[] | Exclusion list |

> **Note on coverage_amount:** For Haftpflicht, record the Deckungssumme. For Teilkasko/Vollkasko, `coverage_amount` is null because payout is based on vehicle repair/replacement cost, not a fixed sum.

---

## Keyword Map

| German Term | English | Notes |
|-------------|---------|-------|
| Kfz-Versicherung | Motor vehicle insurance | General term |
| Haftpflicht | Third-party liability | Mandatory tier |
| Teilkasko | Partial comprehensive | Named-peril own-vehicle coverage |
| Vollkasko | Full comprehensive | All-risk own-vehicle coverage |
| Schadenfreiheitsklasse (SF-Klasse) | No-claims class | Discount factor |
| Typklasse | Vehicle risk class | Model-based rating |
| Regionalklasse | Regional risk class | District-based rating |
| Selbstbeteiligung | Deductible | Per-claim excess |
| Deckungssumme | Coverage limit | For Haftpflicht tier |
| Wildunfall | Wildlife collision | Animal impact coverage |
| Marderbiss | Marten bite | Wiring damage coverage |
| Fahrerschutz | Driver protection | Covers insured driver's injury |
| Schutzbrief | Breakdown rider | Roadside assistance add-on |
| Hauptfalligkeit | Renewal date | Usually 01.01 for Kfz |
| Kundigungsfrist | Notice period | Usually 1 month (by 30.11 for 01.01 renewal) |

---

## Cancellation Rules

**Standard Kundigungsfrist:** 1 month — most Kfz policies renew on 01.01; notice must be received by 30.11.

**Sonderkundigungsrecht triggers:**
- Premium increase — cancel within 4 weeks of notification
- After an insured claim is paid — insurer or insured may cancel within 4 weeks
- After an uninsured claim (insurer declines) — insured may cancel within 4 weeks
- Vehicle sale or deregistration — policy terminates automatically (Versicherung erlischt bei Abmeldung)
- Change of SF-Klasse (after claim regression) — insured has Sonderkundigungsrecht within 4 weeks of notification

**Vehicle sale:** Policy does not automatically transfer. Insured must notify insurer within 14 days. New owner needs own policy.

---

## Sources

- **§1 PflVG (Pflichtversicherungsgesetz):** Mandatory Kfz-Haftpflicht requirement
- **§4 PflVG:** Statutory minimum coverage amounts
- **§6 PflVG:** Criminal penalties for uninsured driving
- **GDV (Gesamtverband der Deutschen Versicherungswirtschaft):** SF-Klasse tables, Regionalklasse — https://www.gdv.de
- **Kraftfahrtbundesamt (KBA):** Typklasse annual publication — https://www.kba.de
- **ADAC:** Independent testing and advice — https://www.adac.de
- **Verbraucherzentrale:** Consumer guidance on Kfz coverage selection
