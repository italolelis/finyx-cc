# Erbpacht (Ground Lease) Detection Guide

## Overview

Erbpacht (Erbbaurecht) is a ground lease arrangement where you own the building but lease the land. This significantly impacts investment returns due to ongoing ground rent (Erbbauzins) and potential lease expiration issues.

**CRITICAL:** Erbpacht with undisclosed costs = AUTOMATIC EXCLUSION

---

## Detection Methods

### 1. Document Search (HIGH Confidence)

Search property documents for these terms:

| Term | Meaning |
|------|---------|
| Erbpacht | Ground lease (common term) |
| Erbbaurecht | Ground lease (legal term) |
| Erbbauzins | Annual ground rent payment |
| Erbbaurechtsvertrag | Ground lease contract |
| Erbbaurechtsgeber | Ground lease grantor |
| Restlaufzeit | Remaining lease term |

**Search command:**
```bash
grep -ri "erbpacht\|erbbaurecht\|erbbauzins" "properties/[location]/" 2>/dev/null
```

### 2. Price List Analysis (MEDIUM Confidence)

Check the price breakdown in Excel/PDF price lists:

| Indicator | Normal | Erbpacht Suspected |
|-----------|--------|-------------------|
| Grundstücksanteil (land share) | 20-35% | < 15% |
| Gebäudeanteil (building share) | 65-80% | > 85% |
| Land value line item | Present | Missing or minimal |

**Red flag:** If building portion is >85% of purchase price, Erbpacht is likely.

### 3. Web Research (MEDIUM Confidence)

Search patterns:
- `"[development name] [location] Erbpacht"`
- `"[development name] Erbbaurecht"`
- `"[developer name] ground lease"`
- `"[location] church land"` (churches often use Erbpacht)

**Common Erbpacht grantors:**
- Churches (Kirche, Bistum, Erzbistum)
- Municipalities (Stadt, Gemeinde)
- Foundations (Stiftung)
- State entities (Land, Bund)

### 4. Developer History

Some developers frequently use Erbpacht. Check:
- Previous projects by same developer
- Developer's land acquisition strategy
- Location of development (church/municipal land areas)

---

## Confidence Levels

| Evidence | Confidence | Action |
|----------|------------|--------|
| Explicit "Erbpacht" in documents | HIGH | Verify costs |
| Erbbauzins amount stated | HIGH | Calculate impact |
| Land share < 15% in price list | MEDIUM | Investigate further |
| Developer uses Erbpacht elsewhere | LOW | Ask directly |
| No evidence found | - | Proceed with caution |

---

## Erbpacht Status Classification

### ✅ NO ERBPACHT DETECTED

- No documentary evidence found
- Land share in normal range (20-35%)
- Web search shows no Erbpacht mentions
- **Action:** PROCEED with analysis

### ⚠️ ERBPACHT SUSPECTED

- Indirect indicators present (low land share, known Erbpacht area)
- No explicit confirmation
- **Action:** VERIFY before proceeding
  - Contact developer/sales directly
  - Request Erbpacht disclosure
  - Check Grundbuch (land registry) if possible

### 🚨 ERBPACHT CONFIRMED

Erbpacht is confirmed. Evaluate based on cost disclosure:

| Costs Disclosed? | Annual Cost | Action |
|------------------|-------------|--------|
| Yes | < €1,000/year | Calculate impact, may proceed |
| Yes | €1,000-3,000/year | Significant impact, recalculate returns |
| Yes | > €3,000/year | Likely EXCLUDE |
| No | Unknown | **EXCLUDE** |

---

## Impact Calculation

If Erbpacht is confirmed with disclosed costs:

### Annual Impact
```
Erbbauzins = €[ANNUAL_AMOUNT]
Monthly impact = Erbbauzins ÷ 12

Adjusted Monthly Cashflow = Original Cashflow - Monthly Erbbauzins
```

### Yield Impact
```
Adjusted Gross Yield = (Annual Rent - Erbbauzins) ÷ Total Price × 100
```

### Long-term Considerations

| Factor | Impact |
|--------|--------|
| Erbbauzins increases | Usually indexed to inflation or land value |
| Lease expiration | Check Restlaufzeit (remaining term) |
| Renewal terms | May require renegotiation |
| Resale | Buyers discount Erbpacht properties |

**Rule of thumb:** Erbpacht properties trade at 10-20% discount vs. freehold.

---

## Action Matrix

| Detection | Costs | Remaining Term | Action |
|-----------|-------|----------------|--------|
| Not detected | N/A | N/A | PROCEED |
| Suspected | Unknown | Unknown | VERIFY first |
| Confirmed | Disclosed, low | > 50 years | PROCEED with adjustment |
| Confirmed | Disclosed, high | > 50 years | Calculate, likely EXCLUDE |
| Confirmed | Disclosed | < 30 years | EXCLUDE (resale risk) |
| Confirmed | NOT disclosed | Any | **EXCLUDE** |

---

## Documentation

When Erbpacht is detected, document in location research:

```markdown
## Erbpacht Status

**Status:** [✅ NO ERBPACHT | ⚠️ SUSPECTED | 🚨 CONFIRMED]

**Evidence:**
- [List all evidence found]

**If Confirmed:**
- Erbbauzins: €[AMOUNT]/year
- Remaining term: [YEARS] years
- Grantor: [CHURCH/MUNICIPALITY/OTHER]

**Impact:**
- Monthly: -€[AMOUNT]
- Yield reduction: -[X]%

**Action:** [PROCEED | VERIFY | EXCLUDE]
**Reason:** [Explanation]
```
