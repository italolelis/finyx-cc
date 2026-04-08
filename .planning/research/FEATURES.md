# Feature Landscape: PKV vs GKV Health Insurance Advisor

**Domain:** German health insurance decision advisory (PKV vs GKV)
**Researched:** 2026-04-06
**Milestone:** v1.2 — `/finyx:insurance` command

---

## Table Stakes

Features a PKV vs GKV decision tool must have. Missing any = advice is incomplete or wrong.

| Feature | Why Expected | Complexity | Profile Data Used |
|---------|--------------|------------|-------------------|
| Versicherungspflichtgrenze eligibility gate | First question: "can I even choose?" — 2026 threshold is €77,400 gross/year. Below it: stop, GKV is mandatory. | Low | `income.gross_annual` |
| GKV cost calculation | Income × (14.6% + Zusatzbeitrag ~2.9%) / 2, capped at BBG (€69,750/yr in 2026). Employer pays half for employees. Self-employed pay full rate. | Low | `income.gross_annual`, `employment.type` |
| PKV indicative cost range | Age + health status → base tariff estimate. Cannot be exact without insurer underwriting, but ballpark is required. Flag as estimate. | Medium | `personal.age`, `health.pre_existing_conditions` |
| Break-even income point | At what gross income does PKV become net-cheaper than GKV after tax deduction? Critical for near-threshold earners. | Low | derived from above |
| Family impact analysis | Familienversicherung: non-working spouse + children covered free in GKV. In PKV each person needs a separate contract. Families with one earner almost always better in GKV. | Medium | `family.partner_employed`, `family.children_count` |
| Long-term cost projection | PKV premiums tied to age and medical inflation (avg +3.1%/yr historically). GKV tied to income + BBG growth (avg +3.8%/yr). 20–40 year horizon often reverses the answer. | High | `personal.age`, `personal.retirement_age` |
| Tax deduction netting for PKV | Basisabsicherung portion is fully deductible as Sonderausgaben with no cap. 2026 change: insurers now transmit data to tax office via ELStAM. Net PKV cost materially lower than gross premium. | Medium | `tax.steuerklasse`, `income.gross_annual` |
| Age-out lock-in warning | After 55, returning to GKV as an employee is nearly impossible by law. Users 50+ must see this risk prominently. | Low | `personal.age` |
| Altersrückstellungen explanation | Mandatory PKV old-age reserves partially offset premium growth at retirement. Without this context, users systematically overestimate PKV retirement cost risk. | Low | narrative only |
| Employer Arbeitgeberzuschuss cap | Employer subsidizes PKV up to the GKV equivalent contribution — not half the actual PKV premium. For expensive tariffs, the employer contribution gap matters. | Low | `employment.type` |
| Beamter redirect | Civil servants use the Beihilfe system (state pays 50–80%) making PKV standard for them. Completely different decision logic. | Low | `employment.type` |
| Pflegeversicherung inclusion | Long-term care insurance is mandatory and often omitted. GKV members pay ~3.4% of income; PKV members pay a separate private Pflegeversicherung tariff. | Low | add to both cost models |

---

## Differentiators

Features that make `/finyx:insurance` better than a generic web calculator, enabled by Finyx's existing profile data and cross-advisor architecture.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Health questionnaire (simplified) | PKV underwriting applies Risikozuschlag (10–50%+ surcharge) or Risikoausschluss (coverage exclusion) for pre-existing conditions. Without this, PKV cost estimates are wrong. | Medium | Collect: chronic conditions flag, mental health treatment in last 5 years flag, BMI range, ongoing medications. Binary flags only — do NOT collect diagnoses. Always recommend Anonyme Risikovoranfrage via licensed broker. |
| Beitragsrückerstattung modeling | PKV refunds 1–4 months of premiums if no claims filed. Healthy low-utilization users recover €500–2,000/yr. GKV has no equivalent. Directly affects effective PKV cost. | Low | Ask: healthcare usage (low/medium/high). Show premium with and without estimated refund. |
| Selbstbeteiligung scenarios | Annual deductible options (€300–€2,000+) reduce PKV premiums 10–30%. Users willing to self-fund routine costs can substantially close the GKV/PKV price gap. | Low | Show delta: premium at €0 / €500 / €1,000 / €2,000 deductible. |
| Anwartschaft advisory for expats | Users temporarily abroad can keep PKV dormant for ~€30–80/month (Anwartschaft) without new underwriting on return. GKV re-entry abroad requires qualifying income again. Critical for the DE+BR expat profile. | Medium | Trigger when `residence.primary_country != DE` or `profile.expat_status = true`. |
| Cross-advisor tax integration | PKV Basisabsicherung deduction reduces taxable income → links to Finyx tax analysis (Sonderausgaben gap) and pension planning (headroom recalculation). | Medium | Reads existing `.finyx/profile.json`; no new data required. Pass finding to `/finyx:insights` as actionable item. |
| Provider research agent | Web-search current tariffs from major providers (AXA, Allianz, Debeka, Ottonova, HUK-Coburg). Return indicative premium ranges, Beitragsrückerstattung terms, digital service quality. | High | Spawns sub-agent with web search. Mandatory disclaimer: not binding quotes. Recommend Anonyme Risikovoranfrage. |

---

## Anti-Features

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Binding PKV premium quotes | Underwriting requires full Gesundheitsprüfung by a licensed broker. Presenting estimates as quotes creates liability and erodes trust. | Label all numbers as indicative ranges. Recommend Anonyme Risikovoranfrage via broker before any switch. |
| PKV provider ranking / "best PKV" list | Highly context-dependent (age, health, region, tariff structure). Ranking implies claims processing quality assessment we cannot do. | Show 3–5 representative providers with key differentiators. No ranking. |
| Check24-style full comparison table | Full insurance marketplace is explicitly out of scope (PROJECT.md). Requires live data feeds we do not have. | Single decision: GKV or PKV verdict + conditions + top 3 action items. |
| Unqualified "switch to PKV" recommendation | Age-out lock-in, pre-existing condition exclusions, and family scenarios can make a PKV switch financially catastrophic. An unqualified recommendation creates real user harm. | Always qualify verdict with explicit conditions. Always recommend licensed broker consultation for final decision. |
| GKV provider comparison | Zusatzbeitrag variance (2.18%–3.4% in 2026) is small; statutory benefits are identical by law. Comparison adds noise. | Mention Zusatzbeitrag range; link to GKV-Zusatzbeitrag.de for current rates. |
| §204 VVG in-PKV tariff switching guidance | Complex broker-assisted process. Out of scope for v1.2. | Mention that in-PKV tariff optimization via §204 VVG exists; recommend broker. |
| Manual premium quotes for specific conditions | Asking users to input their actual PKV quote creates liability when our model diverges. | Use age-bracket ranges. Acknowledge the estimate nature explicitly. |

---

## Feature Dependencies

```
income.gross_annual
    → Versicherungspflichtgrenze gate (hard stop if below €77,400)
    → GKV cost calculation (× combined rate, capped at BBG)
    → Tax deduction value (marginal rate × Basisabsicherung portion)

employment.type
    → Beamter? → redirect to Beihilfe explanation (different system)
    → Selbständig? → full GKV rate (no employer split), different GKV entry rules
    → Employee? → employer Arbeitgeberzuschuss applies

personal.age
    → PKV indicative premium (age-rated)
    → Long-term projection (years to retirement)
    → Age-out lock-in warning trigger (50+)
    → Altersrückstellungen accumulation estimate

health.pre_existing_conditions / health.chronic_medication
    → Risikozuschlag flag → adjust PKV estimate upward
    → Severe cases: recommend checking PKV feasibility via Anonyme Risikovoranfrage before analysis

family.partner_employed + family.children_count
    → Familienversicherung benefit calculation (per non-working dependent: ~€300–500/month saved in GKV vs PKV)

residence.primary_country / expat_status
    → Anwartschaft advisory (expat edge case)

health.healthcare_usage
    → Beitragsrückerstattung estimate
    → Selbstbeteiligung recommendation

Basisabsicherung deduction (output)
    → feeds /finyx:tax (Sonderausgaben unused gap)
    → feeds /finyx:insights (net insurance cost after tax)
```

---

## New Profile Fields Required

Confirm existence of these fields in `.finyx/profile.json`; collect missing ones at command start.

| Field | Type | Notes |
|-------|------|-------|
| `health.current_insurance` | enum: `GKV` / `PKV` | Are they already in PKV? Changes the advice direction. |
| `health.pre_existing_conditions` | boolean | Binary only. Do not collect diagnoses. |
| `health.chronic_medication` | boolean | Proxy for underwriting risk. |
| `health.healthcare_usage` | enum: `low` / `medium` / `high` | For Beitragsrückerstattung estimate. |
| `employment.type` | enum: `employee` / `self_employed` / `beamter` / `student` | Beamter is a completely separate path. |
| `family.partner_employed` | boolean | Familienversicherung gate. |
| `family.children_count` | integer | Per-child PKV cost multiplier vs GKV zero marginal cost. |
| `personal.retirement_age` | integer | Long-term projection endpoint. |

---

## MVP Build Order for v1.2

1. Eligibility gate — Versicherungspflichtgrenze with 2026 threshold (€77,400)
2. GKV cost calculation — BBG cap, Zusatzbeitrag, employer split, Pflegeversicherung included
3. PKV indicative range — age-bracket base with health-flag surcharge
4. Family impact — Familienversicherung zero-cost vs per-head PKV premium
5. Tax deduction netting — Basisabsicherung as Sonderausgaben; show gross vs net PKV cost
6. Long-term projection — 10/20/30-year cost trajectory
7. Beamter detection — immediately redirect to Beihilfe explanation

Defer to future milestone:
- Provider research agent (web search freshness risk, high complexity)
- Anwartschaft advisory (expat edge case, lower frequency)
- §204 in-PKV tariff optimization

---

## German Insurance Terminology Reference

| Term | Meaning | 2026 Value |
|------|---------|------------|
| Versicherungspflichtgrenze / JAEG | Income threshold above which employees may choose PKV | €77,400 gross/yr |
| Beitragsbemessungsgrenze (BBG) | GKV contribution calculated on income up to this cap | €69,750/yr |
| Allgemeiner Beitragssatz | Statutory GKV rate split equally between employee and employer | 14.6% |
| Zusatzbeitrag | Provider-specific GKV surcharge; varies by fund | Avg 2.9%, range 2.18%–3.4% |
| Familienversicherung | Free GKV co-insurance for non-working spouse and children | Zero marginal cost in GKV |
| Altersrückstellungen | Mandatory PKV reserves accumulating during working years to subsidize retirement premiums | Structural PKV advantage |
| Beitragsrückerstattung | PKV premium refund for claim-free years | Typically 1–4 months |
| Selbstbeteiligung | Annual deductible; reduces premium 10–30% | €300–€2,000+ options |
| Risikozuschlag | Premium surcharge for pre-existing conditions | Typically 10–50%+ |
| Risikoausschluss | Coverage exclusion for a specific condition | Eliminates PKV benefit for that condition |
| Anwartschaft | Dormant PKV policy preserving underwriting status during absence from Germany | ~€30–80/month |
| Gesundheitsprüfung | Health questionnaire required for PKV enrollment | Mandatory underwriting |
| Anonyme Risikovoranfrage | Anonymous pre-check with insurer to assess underwriting outcome without creating an official record | Risk mitigation strategy |
| Beihilfe | State subsidy for civil servants covering 50–80% of healthcare costs | Separate system entirely |
| Pflegeversicherung | Mandatory long-term care insurance | GKV ~3.4% of income |
| §204 VVG | Legal right to switch to comparable tariff within same PKV insurer without new health check | In-PKV optimization |

---

## Sources

- Versicherungspflichtgrenze 2026: https://www.myhealthcarebroker.com/blog/jahresarbeitsentgeltgrenze-2026-private-health-insurance
- BBG / Zusatzbeitrag 2026: https://www.thegoodbroker.de/post/changes-in-gkv-contributions-2026
- Familienversicherung impact: https://www.how-to-germany.com/health-insurance/private-health-insurance/families-children/
- PKV long-term cost trajectory: https://germanpedia.com/private-health-insurance-cost-development/
- Altersrückstellungen mechanics: https://www.ottonova.de/en/v/private-health-insurance/private-health-insurance-premiums-in-old-age
- Tax deductibility 2026 ELStAM change: https://die-finanzpruefer.de/steuern/steuererleichterung-pkv-arbeitnehmer/
- Risikozuschlag / Gesundheitsprüfung: https://pkv-welt.de/risikozuschlag/
- Anwartschaft for expats: https://www.privat-patienten.de/lexikon/begriff/auslandsaufenthalt-dauerhaft/
- Beitragsrückerstattung: https://www.versicherungsantrag24.de/pkv-beitragsrueckerstattung-fluch-oder-segen/
- Age-out lock-in risk: https://financeforexpats.de/news/why-you-should-not-switch-to-private-health-insurance-in-germany
- Is PKV worth it 2026: https://feather-insurance.com/blog/private-health-worth-it
