---
tax_year: 2025
country: germany
domain: investment-tax
last_updated: 2026-04-06
source: BMF, Bundesbank, EStG, InvStG 2018
---

# German Investment Taxation Reference

This document covers the key German investment tax rules applicable to private investors — ETFs, stocks, dividends, and capital gains. It is loaded by `/finyx:tax` to provide accurate, year-stamped guidance.

> **Tax year notice:** This document reflects rules for tax year 2025. The `/finyx:tax` command will warn you if this does not match the current year. Verify all figures against official BMF/Finanzamt sources before acting.

---

## 1. Steuerklassen I–VI

German tax classes (Steuerklassen) determine how employment income is withheld at source by the employer. There are six classes:

| Class | Who it applies to | Key characteristics |
|-------|------------------|---------------------|
| **I** | Single, divorced, widowed (no qualifying children) | Standard Grundfreibetrag; default for most singles |
| **II** | Single parent with custody | Entlastungsbetrag für Alleinerziehende (~4,260 EUR/year relief) |
| **III** | Married — higher-earning partner | Lower withholding; must be paired with Class V for lower earner |
| **IV** | Married — equal earners (default for married couples) | Each partner taxed as if individual; balanced withholding |
| **IV with factor** | Married — moderate income gap | Factor method splits withholding more accurately; reduces large year-end adjustments |
| **V** | Married — lower-earning partner | Paired with Class III; higher withholding, no personal allowances |
| **VI** | Second or additional job | No Freibetrag at all; very high withholding rate |

### Recommendation logic

- **Single, no children:** Class I
- **Single parent with custody:** Class II (file with Finanzamt to activate)
- **Married with large income gap (>3:1):** Class III for the higher earner, Class V for the lower earner
- **Married with moderate income gap:** Class IV with factor method — reduces surprises at year-end
- **Married, similar incomes:** Class IV / IV (symmetric withholding)
- **Second job:** Class VI applies automatically

### Important: Steuerklassen do NOT affect capital gains tax

> Steuerklassen affect employment income withholding only. Capital gains are taxed at the flat 25% Abgeltungssteuer rate regardless of tax class. However, your tax class affects your marginal rate, which determines whether you qualify for the Günstigerprüfung (see Section 2).

---

## 2. Abgeltungssteuer (Capital Gains Flat Tax)

Since 2009, German investment income is subject to a flat withholding tax called Abgeltungssteuer.

### Rates

| Component | Rate | Calculation |
|-----------|------|-------------|
| Abgeltungssteuer | 25.000% | Flat rate on all capital income |
| Solidaritätszuschlag | 1.375% | 5.5% of 25% |
| **Effective rate (no church tax)** | **26.375%** | |
| + Church tax (8%, e.g. Bavaria, BW) | +1.444% | 8% of 25% adjusted | 
| **Effective rate (church tax 8%)** | **27.819%** | |
| + Church tax (9%, other states) | +1.620% | 9% of 25% adjusted |
| **Effective rate (church tax 9%)** | **27.995%** | |

### What is subject to Abgeltungssteuer

- Dividends from stocks and funds
- Capital gains from selling stocks, ETFs, bonds
- Interest income (savings, fixed-term deposits, bonds)
- Vorabpauschale (annual advance tax on accumulating funds)

### Günstigerprüfung (Favorability Check)

If your marginal income tax rate is **below 26.375%**, you are paying more tax on investments than you need to. File Anlage KAP with the Günstigerprüfung option checked, and the Finanzamt will recalculate your capital gains at your lower marginal rate and refund the difference.

**Relevant for:** Students, part-time workers, low-income years, early retirement.

> For most full-time employed investors earning above ~€20,000/year, the marginal rate exceeds 26.375%, so Günstigerprüfung does not help.

### Anlage KAP: When to file

| Situation | Action |
|-----------|--------|
| All income at German brokers, Freistellungsauftrag set | Usually no filing needed — broker handles everything |
| Foreign broker (Trading212, IBKR, etc.) | Must file Anlage KAP — broker doesn't withhold German tax |
| Cross-broker losses to offset | File to transfer loss certificates (Verlustbescheinigung) |
| Günstigerprüfung eligible | File and check the box — may result in refund |
| Freistellungsauftrag forgotten | File to reclaim over-withheld tax |

---

## 3. Sparerpauschbetrag (Saver's Allowance)

Every German tax resident receives an annual tax-free allowance on investment income.

### Current allowance (since tax year 2023)

| Filing status | Annual allowance |
|--------------|-----------------|
| Single | **1,000 EUR** |
| Married (joint assessment) | **2,000 EUR** |

> **Warning: The old 801 EUR / 1,602 EUR limits applied through tax year 2022. Do NOT use those figures for 2023 onwards.**

### Freistellungsauftrag (Exemption Order)

To use the allowance, you must file a Freistellungsauftrag with each broker. Key rules:

- **Separate filing per broker** — each receives its own allocation
- **Sum must not exceed your total allowance** — splitting 1,000 EUR across multiple brokers is permitted, but the total must stay at or below 1,000 EUR (single) or 2,000 EUR (married)
- **Exceeding the allowance is illegal** under §44a EStG and triggers a Finanzamt query — the broker does not enforce this for you across brokers
- **Applies to all investment income** — dividends, interest (savings, fixed deposits), capital gains, and Vorabpauschale all count against the allowance

### Cross-broker strategy

Allocate your Freistellungsauftrag proportionally to where you expect to earn the most capital income:

```
Total allowance (single):      1,000 EUR
- ING savings interest:           50 EUR  → Freistellungsauftrag ING: 50 EUR
- Trade Republic dividends:       600 EUR  → Freistellungsauftrag TR:  600 EUR
- Trading 212 dividends:          200 EUR  → Freistellungsauftrag T212: 200 EUR
────────────────────────────────────────
Remaining unused allowance:       150 EUR
```

**Tips:**
1. Concentrate allocation at the broker with the most expected income — simpler and lower risk of over-allocation
2. Accumulating ETFs also consume the allowance via Vorabpauschale — factor this in
3. Check utilization mid-year via broker dashboards and reallocate if needed
4. Savings account interest counts against the 1,000 EUR — don't forget it

---

## 4. Teilfreistellung (Partial Exemption for Funds)

Investment funds (ETFs, mutual funds) receive a partial exemption from Abgeltungssteuer. This is a permanent feature of the InvStG 2018 regime.

### Exemption rates by fund type

| Fund type | Definition | Exemption | Effective tax rate* |
|-----------|-----------|-----------|---------------------|
| Equity funds (Aktienfonds) | >50% stocks | **30%** | ~18.46% |
| Mixed funds (Mischfonds) | 25–50% stocks | **15%** | ~22.42% |
| Bond/other funds | <25% stocks | **0%** | 26.375% |
| Real estate funds — domestic (>50% DE property) | — | **60%** | ~10.55% |
| Real estate funds — foreign (>50% non-DE property) | — | **80%** | ~5.28% |

*Effective tax rates shown without church tax.

### What Teilfreistellung applies to

Teilfreistellung applies to **all taxable events from funds**:
- Distributions (dividends paid out by a distributing fund)
- Capital gains on sale of fund units
- Vorabpauschale (advance tax on accumulating funds)

### Critical: Teilfreistellung does NOT apply to individual stocks

> Teilfreistellung applies to investment funds (ETFs and mutual funds) ONLY — not to dividends or capital gains from individual stocks. A MSCI World ETF gets 30% exemption; a dividend from Apple shares does not.

This is a common filing error — see Section 6 (Anlage KAP) for correct treatment when using foreign brokers.

---

## 5. Vorabpauschale (Annual Advance Tax on Accumulating Funds)

Accumulating (thesaurierend) ETFs do not distribute dividends — they reinvest gains internally. To prevent indefinite tax deferral, Germany charges an annual minimum tax called the Vorabpauschale.

### Formula

```
Basisertrag      = Fund value on Jan 1 × Basiszins × 0.70
Vorabpauschale   = min(Basisertrag, actual fund gain in calendar year)
```

If the fund lost value during the year, or if the fund gained less than the Basisertrag, the Vorabpauschale is reduced accordingly (but never negative).

### Applying Teilfreistellung

After calculating the Vorabpauschale, apply Teilfreistellung before computing tax:

```
Taxable Vorabpauschale (equity ETF) = Vorabpauschale × (1 - 0.30) = Vorabpauschale × 0.70
Tax due = Taxable Vorabpauschale × 26.375%  (or 27.995% with church tax)
```

### Current Basiszins values

| Tax year | Basiszins | Vorabpauschale deducted |
|----------|-----------|------------------------|
| 2025 | **2.29%** | January 2026 |
| 2026 | **3.20%** | January 2027 |

> **Verify 2026 Basiszins:** The BMF and Deutsche Bundesbank publish the Basiszins in early January each year. The 3.20% figure shown here should be confirmed against the current BMF publication before filing.

### Practical example (2025 tax year)

```
ETF value Jan 1, 2025:        100,000 EUR
Basiszins 2025:                    2.29%
Basisertrag:    100,000 × 2.29% × 0.70  =  1,603 EUR
ETF gain in 2025 (actual):         8,000 EUR
Vorabpauschale:  min(1,603, 8,000)       =  1,603 EUR

Equity ETF → Teilfreistellung 30%:
Taxable:       1,603 × 0.70             =  1,122.10 EUR
Tax:           1,122.10 × 26.375%       =    295.96 EUR
```

### Practical impact

- The broker automatically deducts the Vorabpauschale tax in **January** for the prior year
- **Keep a cash buffer** in your broker account — if insufficient cash, the broker may sell fund units
- The Vorabpauschale is deducted from the Sparerpauschbetrag first (reduces how much of the allowance is left for other income)
- The taxed Vorabpauschale **increases your cost basis** — you will not be double-taxed when you later sell

### Accumulating vs distributing ETFs and the Vorabpauschale

| ETF type | Dividends paid out | Vorabpauschale applies | Annual tax event |
|----------|-------------------|----------------------|------------------|
| Distributing (ausschüttend) | Yes | Only if distribution < Basisertrag | Dividends taxed on payment date |
| Accumulating (thesaurierend) | No | Yes | January (prior year's Vorabpauschale) |

> **Note:** For accumulating ETFs held at a **foreign broker** (Trading212, IBKR, etc.), the broker does NOT withhold the Vorabpauschale — you must calculate it manually and declare it on Anlage KAP-INV.

---

## 6. Anlage KAP Line Mapping (Foreign Brokers)

German brokers handle withholding automatically. Foreign brokers do not withhold German tax — all income must be declared manually on Anlage KAP.

### Key lines for foreign broker income (Lines 18–26)

| Line | Content |
|------|---------|
| **19** | Total capital income without German withholding (see calculation below) |
| **22** | Losses realized during the year |
| **23** | Losses from stock sales specifically (Aktienverlusttopf) |
| **41** | Foreign withholding tax credited against German tax (e.g., US 15% WHT) |

### Calculating Line 19 correctly

```
Stock dividends (fully taxable, no Teilfreistellung)   +X,XXX.XX
+ Share lending income (fully taxable)                 +   XX.XX
+ Equity ETF distributions × 0.70 (30% TFS applied)   +  XXX.XX
+ Bond ETF distributions × 1.00 (0% TFS)              +   XX.XX
+ Stock gains from closed positions (fully taxable)    +  XXX.XX
+ Equity ETF gains × 0.70 (30% TFS applied)           +  XXX.XX
- FX conversion fees                                   -   XX.XX
= Line 19 total                                        X,XXX.XX
```

> Remember: apply Teilfreistellung to ETF income before entering on Line 19. Do NOT apply it to individual stock dividends or stock gains.

---

## 7. Tax Calendar

| Month | Event | What to do |
|-------|-------|------------|
| **January** | Vorabpauschale deducted by broker | Ensure sufficient cash in all broker accounts by late January |
| **Ongoing** | Freistellungsauftrag monitoring | Check broker dashboards; reallocate if one broker uses more than expected |
| **Before Dec 15** | Verlustbescheinigung deadline | Request from each broker if you want to use their losses at another broker |
| **December** | Year-end review | Harvest unrealized losses if beneficial (no wash sale rule in Germany) |
| **March–May** | Steuererklärung season | File Anlage KAP if you used a foreign broker, have cross-broker losses, or want Günstigerprüfung |
| **Annual** | Freistellungsauftrag review | Adjust allocations based on actual income per broker from prior year |

> **Tax-loss harvesting:** Germany has no wash sale rule — you can sell a losing position and immediately repurchase it to realize the loss for tax purposes. Detailed tax-loss harvesting guidance is a planned future feature in Finyx v2.

---

## 8. Important Notes and Common Pitfalls

### Pre-2018 ETF taxation is superseded

The InvStG 2018, effective January 1, 2018, fundamentally changed how ETFs are taxed in Germany. All pre-2018 special rules (e.g., for swap-based ETFs, the old "ausschüttungsgleiche Erträge" regime) are fully superseded. If you encounter references to pre-2018 rules, disregard them.

### Sparerpauschbetrag: do not use the old 801 EUR limit

The Sparerpauschbetrag was raised from 801 EUR (single) / 1,602 EUR (married) to **1,000 EUR / 2,000 EUR effective January 1, 2023**. Tax software, forum posts, and articles written before 2023 may still quote the old limit. Always use 1,000 / 2,000 EUR for tax years 2023 onwards.

### Irish/Luxembourg ETF domicile preference

For German investors, ETFs domiciled in Ireland or Luxembourg are generally the most tax-efficient:
- Irish-domiciled ETFs (UCITS): no additional withholding on distributions to German residents
- US-domiciled ETFs: generally not available to EU investors (MiFID II KIID requirement); if held via grandfathered positions, US 15% WHT applies and is creditable

### Church tax on capital gains

Church tax (8% or 9% depending on state) also applies to Abgeltungssteuer on investment income for registered church members. German brokers are required to automatically withhold church tax on investment income for church members — if your broker does not have your church tax status, you will need to file Anlage KAP to settle the difference.

---

*Source: BMF, §32d EStG, §2 InvStG 2018, Deutsche Bundesbank (Basiszins), §44a EStG. Verify current-year Basiszins and tax class thresholds against official BMF publications.*
