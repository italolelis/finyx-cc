---
tax_year: 2025
country: germany
domain: health-insurance
last_updated: 2026-04-06
source: "GKV-Spitzenverband, §223 SGB V, §241 SGB V, §57 SGB XI, §6 SGB V, §10 EStG, BaFin"
---

# German Health Insurance Reference

> **Tax year notice:** This document reflects rules for tax year 2025 (JAEG €77,400, BBG €69,750/yr). The `/finyx:insurance` command will warn you if this does not match the current year. JAEG, BBG, Zusatzbeitrag average, and Pflegeversicherung rates change annually — verify before acting.

---

## 1. GKV (Gesetzliche Krankenversicherung)

### 1.1 Contribution Formula

**Path 1: Employee (employed, GKV member)**

```
monthly_gross_capped = min(gross_monthly_income, 5812.50)   # BBG monthly cap
gkv_employee = monthly_gross_capped × (0.073 + zusatzbeitrag / 2)
pv_employee  = monthly_gross_capped × pv_rate_employee      # 0.019 (with children) or 0.025 (childless 23+)
total_employee_monthly = gkv_employee + pv_employee
```

**Path 2: Self-Employed (freiwillige GKV)**

Self-employed persons pay both employee and employer shares — no Arbeitgeberzuschuss applies:

```
monthly_gross_capped = min(gross_monthly_income, 5812.50)
gkv_self_employed = monthly_gross_capped × (0.146 + zusatzbeitrag)
pv_self_employed  = monthly_gross_capped × pv_rate_full     # combined employee + employer rate
total_gkv_self_employed = gkv_self_employed + pv_self_employed
```

**Path 3: Family (Familienversicherung)**

Non-working partner (income < €603/month) and children qualify for free coverage:

```
total_family_gkv = total_gkv_cost_to_employee   # no multiplier; €0 additional premium
```

**Path 4: Beamter**

→ Redirect to Section 6.1 (Beihilfe system; standard GKV formula does not apply)

### 1.2 Beitragsbemessungsgrenze (BBG)

The BBG is the **GKV contribution cap** — income above this threshold is not subject to GKV contributions.

| Parameter | 2026 Value |
|-----------|-----------|
| BBG annual | €69,750 |
| BBG monthly | €5,812.50 |

> **Important:** The BBG (€69,750) is the CONTRIBUTION CAP for GKV calculations. It is **not** the PKV eligibility threshold. The PKV eligibility threshold (JAEG) is €77,400 for new switchers — see Section 4.1. Never substitute one for the other in formulas.

### 1.3 Zusatzbeitrag

Each statutory insurer charges a fund-specific Zusatzbeitrag on top of the allgemeiner Beitragssatz (base rate: **14.6%**, split equally 7.3% employee / 7.3% employer per §241 SGB V). The GKV-Spitzenverband publishes the average Zusatzbeitrag annually.

| Field | 2026 Value | Notes |
|-------|-----------|-------|
| GKV-Spitzenverband average | 2.9% | Half paid by employee, half by employer |
| Statutory range | 2.18%–4.4% | Fund-specific; fetched live by Phase 10 research agent |
| fallback_rate | 2.9% | Use if live fetch unavailable |
| source_url | https://www.gkv-spitzenverband.de | Official GKV-Spitzenverband publication |

> Phase 10 research agent fetches the user's specific fund rate via WebSearch using `source_url` as the starting point. Use `fallback_rate: 2.9%` if the live fetch fails.

### 1.4 Pflegeversicherung (PV)

Pflegeversicherung (PV) is collected alongside GKV contributions. Rates vary by parental status.

| Status | Employee Rate | Employer Rate | Total |
|--------|--------------|--------------|-------|
| With children | 1.9% | 1.7% | 3.6% |
| Childless 23+ | 2.5% | 1.7% | 4.2% |

**Childless surcharge:** Employees aged 23 or older with no children pay an additional +0.6% (employee share only), bringing their employee rate to 2.5%.

**Multi-child reduction:** For 2 or more children under age 25, the employee rate is reduced by −0.25% per child starting from the second child:

| Number of children under 25 | Employee PV rate |
|-----------------------------|-----------------|
| 0 (childless, 23+) | 2.5% |
| 1 | 1.9% |
| 2 | 1.65% |
| 3 | 1.4% |
| 4 | 1.15% |
| 5+ | 1.0% (minimum 2.4% total including employer) |

> Source: §57 SGB XI. Confidence: HIGH (GKV-Spitzenverband).

### 1.5 Employer Contribution Caps (Arbeitgeberzuschuss for PKV employees)

Employees in PKV receive an employer subsidy capped at the equivalent GKV employer contribution. The employer pays the lower of (a) half the actual PKV+PV premium or (b) the statutory cap.

| Component | 2026 Monthly Cap |
|-----------|-----------------|
| Private Krankenversicherung (KV cap) | €508.59 |
| Private Pflegeversicherung (non-Saxony) | €104.63 |
| Private Pflegeversicherung (Saxony) | €75.56 |
| Combined cap (non-Saxony) | €613.22 |
| Combined cap (Saxony) | €584.15 |

**Formula:**

```
employer_subsidy = min(
  (pkv_monthly + ppv_monthly) / 2,
  508.59 + 104.63   # non-Saxony cap; use 508.59 + 75.56 for Saxony
)

net_pkv_cost_to_employee = pkv_monthly + ppv_monthly - employer_subsidy
```

> Source: §257 SGB V (KV subsidy), §61 SGB XI (PV subsidy). Confidence: HIGH.

---

## 2. PKV (Private Krankenversicherung)

### 2.1 Age-Based Premium Estimation

PKV premiums are calculated actuarially per person. The following ranges are **indicative benchmarks, not quotes** — actual tariffs vary significantly by insurer, benefit scope, and individual health assessment.

| Age at Entry | Monthly PKV (single, healthy, comprehensive coverage) |
|-------------|------------------------------------------------------|
| 25 | €250–350 |
| 30 | €350–500 |
| 35 | €450–650 |
| 40 | €550–800 |
| 45 | €700–1,000 |
| 50 | €900–1,200+ |

> These ranges assume good health (no Risikozuschlag), comprehensive coverage including dental and vision, and a single adult. Pre-existing conditions, chosen tariff level, and deductible options materially affect the actual premium. Phase 10 research agent fetches current tariff ranges via WebSearch.

### 2.2 Altersrückstellungen (Aging Provisions)

PKV premiums increase with age because health costs rise. To prevent unaffordable premiums in old age, insurers are legally required to build up Altersrückstellungen (aging provisions) during working years. These reserves:

- Are funded by a surcharge embedded in every PKV premium
- Belong to the insured in a modified transferability form (portability rules apply since the 2009 PKV reform)
- Reduce the rate of future premium increases compared to the actuarially required amount

> Altersrückstellungen do NOT fully prevent premium increases with age — they moderate them. Expect premiums to continue rising, especially after age 50.

### 2.3 Beitragsrückerstattung (No-Claims Bonus)

Most PKV tariffs refund one to three months' premium if no claims are filed during the year. This effectively reduces the net annual cost:

- Typically 1–3 months of premium refunded for a claims-free year
- Refund is forfeited for the year if any claim is filed (including small claims)
- Optimal strategy: pay minor claims out-of-pocket if below the refund value

### 2.4 Selbstbeteiligung (Deductible)

PKV tariffs offer optional annual deductibles that reduce monthly premiums. Common options: €300, €600, €1,200, €2,000/year. A higher deductible reduces premiums but increases out-of-pocket exposure. The deductible choice also interacts with the Beitragsrückerstattung strategy above.

### 2.5 Long-Term Projection Constants

Use these constants to project PKV vs GKV cost divergence over time.

| Parameter | Conservative | Base | Optimistic |
|-----------|-------------|------|-----------|
| PKV annual premium growth | 5% | 6% | 3% |
| GKV BBG growth (income proxy) | 2% | 3% | 4% |

> **Non-linear growth warning:** The conservative 5% PKV growth assumption significantly understates the current cycle. 2026 saw actual PKV premium increases of 7–12% due to the Krankenhausreform (hospital financing reform). If actual annual increases exceed 8% in two consecutive years, the base-case projection should be recalibrated upward. The historical average (4–8%) does not reflect the structural cost pressures from the 2023–2026 reform period.

**Projection formula:**

```
pkv_in_n_years = pkv_today × (1 + pkv_growth_rate) ^ n
gkv_in_n_years = gkv_today × (1 + bbg_growth_rate) ^ n   # income-tracking proxy
```

---

## 3. Familienversicherung

### 3.1 GKV Free Coverage Rules

Statutory GKV provides free family co-insurance for non-working or low-income dependents under §10 SGB V:

**Non-working partner (GKV Familienversicherung):**
- Income must be below €603/month (2026 limit)
- No additional premium — the member's existing contribution covers the partner
- `total_family_gkv = total_gkv_cost_to_employee` (no multiplier)

**Children:**
- Free co-insurance until age 18 (or age 23 if not employed, or age 25 if in education)
- No income limit applies for children under 18
- `child_gkv_cost = 0` (covered under member's contribution)

### 3.2 PKV Per-Person Costs

In PKV, each family member requires their own separate policy:

| Family member | Indicative Monthly PKV Cost |
|---------------|-----------------------------|
| Partner (age 30–40, healthy) | €300–600 |
| Child (per child) | €150–200 |

**Side-by-side comparison (example: employed member, non-working partner, 2 children):**

| System | Member | Partner | 2 Children | Monthly Total |
|--------|--------|---------|-----------|--------------|
| GKV | ~€380 (income-based) | €0 (Familienversicherung) | €0 | ~€380 |
| PKV | ~€450 (age 35) | ~€400 | ~€350 | ~€1,200 |

> For families with a non-working partner and children, GKV's Familienversicherung provides a large structural cost advantage. The PKV financial advantage for families requires very high income and/or significant tax benefit from §10 EStG deduction (see Section 5).

---

## 4. Eligibility Thresholds & Risk Tiers

### 4.1 JAEG and BBG Thresholds

Germany uses two distinct income thresholds in the health insurance context. These are different numbers for different purposes — **never conflate them**.

| Threshold | 2026 Value | Purpose | Legal Basis |
|-----------|-----------|---------|------------|
| JAEG (new switchers) | €77,400 gross/year | PKV eligibility — employee must exceed this threshold for at least three consecutive years to switch from GKV to PKV | §6 SGB V |
| JAEG (existing PKV holders) | €69,750/year (= BBG) | Existing PKV members who drop below this threshold in a given year must return to GKV — different threshold than new switchers | §6 SGB V |
| BBG | €69,750/year (€5,812.50/month) | GKV contribution cap — income above BBG not subject to GKV contributions | §223 SGB V |

> The BBG (€69,750) serves double duty: it is both the contribution cap (Section 1.2) AND the lower threshold for existing PKV holders. The JAEG for new switchers (€77,400) is higher and distinct. These two values must never appear in the same formula — one governs contributions, the other governs eligibility transitions.

### 4.2 3-Tier PKV Risk Model

PKV insurers assess pre-existing conditions and calculate individual risk surcharges. The following model approximates the surcharge tiers. Flags are binary — the questionnaire asks yes/no only (no diagnosis details, no severity modifiers — GDPR Art. 9 compliant).

**Binary health flag list (~15 conditions):**

1. Diabetes (any type)
2. Hypertension (high blood pressure)
3. Asthma or COPD
4. Back or spinal conditions (chronic)
5. Joint conditions (osteoarthritis, rheumatism)
6. Thyroid disorders (hypothyroidism, hyperthyroidism)
7. Allergies (severe, requiring ongoing treatment)
8. Sleep apnea (diagnosed)
9. Elevated BMI (>30)
10. Regular medication (daily prescription drugs)
11. Previous surgery (within last 5 years)
12. Previous hospitalization (within last 5 years)
13. Chronic skin conditions (psoriasis, eczema requiring treatment)
14. Gastrointestinal conditions (IBS, Crohn's, ulcerative colitis)
15. Migraines or chronic headaches (recurring, treated)

**Risk tier calculation:**

```
flag_count = sum(binary_health_flags)   # each flag: 0 or 1

if flag_count == 0:
    risk_tier    = "Tier 0 — Standard rate"
    surcharge    = "0%"
elif flag_count <= 2 and no_high_rejection_flag:
    risk_tier    = "Tier 1 — Minor conditions"
    surcharge    = "10–25%"
else:   # 3+ flags OR any high-rejection flag present
    risk_tier    = "Tier 2 — Multiple or high-risk conditions"
    surcharge    = "30–50%+"

pkv_estimated = pkv_base_bracket × (1 + surcharge_midpoint)
```

| Risk Tier | Condition | Estimated Surcharge |
|-----------|-----------|-------------------|
| Tier 0 | 0 flags | 0% |
| Tier 1 | 1–2 minor controlled conditions, no high-rejection flag | 10–25% |
| Tier 2 | 3+ flags OR any high-rejection flag | 30–50%+ |

### 4.3 Rejection-Risk Callout

> **High rejection probability advisory:** The following conditions frequently result in outright PKV rejection or surcharges that make PKV economically non-viable. These are NOT tiered — they are categorical high-rejection flags. When any of the following apply, the `/finyx:insurance` command must alert the user prominently before running cost projections:
>
> - Mental health history (depression, anxiety disorder, psychotherapy within last 5 years)
> - HIV/AIDS
> - Active cancer or remission under 5 years
> - Cardiac events (heart attack, stroke — any history)
> - Autoimmune disorders (multiple sclerosis, Crohn's disease, lupus, rheumatoid arthritis at disease-modifying therapy level)
>
> Users in these categories should assume PKV is likely unavailable at standard rates and focus the analysis on GKV optimisation instead.

---

## 5. §10 EStG Deduction (Basisabsicherung)

PKV premiums for basic healthcare coverage (Basisabsicherung) are deductible from taxable income under §10 EStG. This deduction partially offsets the higher PKV premiums.

### 5.1 Employee Cap

- **Annual deduction cap:** €1,900
- Applies to employees and civil servants receiving employer subsidy or Beihilfe

### 5.2 Self-Employed Cap

- **Annual deduction cap:** €2,800
- Applies to self-employed persons and freelancers (no employer subsidy received)

### 5.3 ELStAM Electronic Reporting (2026 Change)

From 2026, health insurers transmit Basisabsicherung contribution data directly to the Bundeszentralamt für Steuern (BZSt) via ELStAM. Employees no longer need to self-report their PKV Basisabsicherung on their tax return — the data is pre-populated. Self-employed persons still file via Anlage Vorsorgeaufwand.

**Tax benefit calculation:**

```
basisabsicherung_portion = pkv_monthly × 0.85   # conservative estimate; actual range 70–90%
annual_deductible = min(basisabsicherung_portion × 12, cap)
  # cap: €1,900 for employees, €2,800 for self-employed

annual_tax_benefit = annual_deductible × marginal_rate
net_pkv_monthly    = pkv_monthly - (annual_tax_benefit / 12)
```

**Example (employee, marginal rate 42%, PKV €600/month):**

```
basisabsicherung_portion = 600 × 0.85 = 510 EUR/month
annual_deductible = min(510 × 12, 1900) = min(6120, 1900) = 1900 EUR
annual_tax_benefit = 1900 × 0.42 = 798 EUR
net_pkv_monthly = 600 - (798 / 12) = 600 - 66.50 = 533.50 EUR
```

> The §10 EStG cap of €1,900 is quickly reached for most PKV policies. High earners with a €42% marginal rate still save only ~€67/month from the deduction — the cap limits the benefit significantly.

---

## 6. Special Cases

### 6.1 Beamter / Beihilfe Redirect

Civil servants (Beamte) are covered by the Beihilfe system: the employing state authority pays 50–80% of documented healthcare costs directly (the exact percentage depends on family status and state). Standard GKV/PKV cost models do **NOT** apply to Beamte — both the contribution formula (Section 1.1) and the PKV premium comparison (Section 2) overstate costs for Beihilfe recipients.

When `employment.type == "beamter"`, the `/finyx:insurance` command must redirect to Beihilfe guidance rather than running the standard GKV/PKV cost comparison. Full Beamter model is deferred to v1.3 (BEAM-01).

### 6.2 Expat Anwartschaft

German residents temporarily relocating abroad face a gap in both GKV and PKV coverage:

**GKV:** Statutory insurance is suspended while abroad (>3 months outside the EU). Re-entry is possible on return but requires a qualifying employment event.

**Anwartschaftsversicherung (PKV):** Existing PKV holders can maintain their tariff at a significantly reduced "dormant" premium during temporary relocation. This preserves the age-based premium lock and the right to re-activate without new health underwriting.

**EU portability:** Within the EU/EEA, EHIC (European Health Insurance Card) provides emergency coverage; ongoing healthcare requires coordination between systems. Private supplemental coverage is usually still advisable.

**Non-EU gaps:** Outside the EU, neither GKV nor standard PKV provides comprehensive coverage. Travel health insurance or international PKV top-up is required for gaps exceeding 3–6 months.

> For expats planning temporary relocation, the Anwartschaft option is usually the correct strategy for existing PKV holders — it is significantly cheaper than cancelling and re-entering PKV at a later age.

### 6.3 Age-55 Lock-In Warning (§6 Abs. 3a SGB V)

> **WARNING — Near-Irreversible Decision:**
>
> Under §6 Abs. 3a SGB V, insured persons who have been in PKV for 10 or more years and are aged 55 or older **cannot return to GKV** even if their income falls below the JAEG threshold. The only exceptions are very narrow: becoming unemployed (ALG I) with certain conditions, or specific life events such as marriage to a GKV member where the income immediately drops below threshold.
>
> **Practical implication:** A self-employed person in PKV at age 56 whose business income declines is effectively locked out of GKV regardless of income level. This is an irreversible system selection at that age.
>
> Phase 10 and Phase 11 commands should check `profile.age` and emit a prominent age-55 lock-in warning when the user is 50 or older and currently in PKV, or when the user is 50 or older and considering switching to PKV.

---

*Source: GKV-Spitzenverband (Zusatzbeitrag average, BBG, PV rates), §223 SGB V (BBG), §241 SGB V (base rate), §57 SGB XI (PV rates), §6 SGB V (JAEG thresholds), §257 SGB V (employer KV subsidy), §61 SGB XI (employer PV subsidy), §10 EStG (Basisabsicherung deduction), §6 Abs. 3a SGB V (age-55 lock-in), BaFin (PKV regulation). Verify BBG, JAEG, Zusatzbeitrag average, and PV rates annually via GKV-Spitzenverband publication.*
