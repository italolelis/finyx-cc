# Location Research: [LOCATION]

**Researched:** [DATE]
**Status:** [ACTIVE | EXCLUDED]
**Exclusion Reason:** [if excluded, otherwise remove this line]

---

## Development Overview

| Item | Details |
|------|---------|
| Name | [DEVELOPMENT_NAME] |
| Developer | [DEVELOPER_NAME] |
| Address | [FULL_ADDRESS] |
| City | [CITY] |
| Size | [NUM_UNITS] units |
| Completion | [EXPECTED_DATE] |
| Concept | [e.g., Urban living, Car-free, Family-oriented] |

### Project Description

[Brief description of the development - building type, architecture, amenities, target market]

---

## Erbpacht Status

**Status:** [✅ NO ERBPACHT DETECTED | ⚠️ SUSPECTED | 🚨 CONFIRMED]

### Evidence

| Source | Finding |
|--------|---------|
| Document search | [Result] |
| Price list analysis | Land share: [X]% |
| Web search | [Result] |

### If Confirmed

| Item | Value |
|------|-------|
| Erbbauzins | €[AMOUNT]/year |
| Remaining Term | [YEARS] years |
| Grantor | [CHURCH/MUNICIPALITY/OTHER] |
| Indexed to | [INFLATION/LAND VALUE/FIXED] |

### Action

**Decision:** [PROCEED | VERIFY | EXCLUDE]
**Reason:** [Explanation]

---

## Public Transport

| Item | Details |
|------|---------|
| Nearest Stop | [STOP_NAME] |
| Distance | [X] meters ([Y] min walk) |
| Lines | [e.g., Tram 1, 3 / Bus 42, 56 / S-Bahn S1] |
| To City Center | [X] minutes |
| Frequency (Peak) | Every [X] minutes |
| Weekend Service | [YES/LIMITED/NO] |
| Night Service | [YES/LIMITED/NO] |

**Network Quality:** [EXCELLENT | GOOD | MODERATE | POOR]

**Assessment:** [Detailed transport assessment]

---

## Parking Necessity

**Development Concept:** [Standard | Car-reduced | Car-free]

**Verdict:** [ESSENTIAL | IMPORTANT | PREFERRED | OPTIONAL]

**Reasoning:**
[Explanation based on transport quality + development concept + location type]

**Implications for Analysis:**
- [ ] Only recommend units with parking
- [ ] Prioritize parking units but consider non-parking if yield is significantly better
- [ ] Parking adds value but not essential for rentability

---

## Market Conditions

| Metric | Value | Source |
|--------|-------|--------|
| Housing Deficit/Surplus | [VALUE] units | [SOURCE] |
| Price Trend (5yr) | [+/-X]%/year | [SOURCE] |
| Current Price Level | €[X]/m² | [SOURCE] |
| Rental Demand | [HIGH/MODERATE/LOW] | [SOURCE] |
| Vacancy Rate | [X]% | [SOURCE] |

### Economic Factors

| Factor | Details |
|--------|---------|
| Major Employers | [LIST] |
| Universities | [LIST] |
| Population Trend | [GROWING/STABLE/DECLINING] |
| Infrastructure Projects | [Any major projects nearby] |

### Market Assessment

[Summary of market conditions and investment outlook for the location]

---

## Documents Found

| File | Type | Notes |
|------|------|-------|
| [filename.xlsx] | Price List | [X] units listed |
| [filename.pdf] | Exposé | Marketing brochure |
| [filename.pdf] | Calculation | Developer calculation example |
| [filename.pdf] | Floor Plans | Building [X] |

### Document Quality

- [ ] Price list available with all unit data
- [ ] Rent estimates provided
- [ ] Financing terms specified
- [ ] Construction timeline clear

---

## Red Flags

| Flag | Severity | Details |
|------|----------|---------|
| [e.g., Erbpacht undisclosed] | HIGH | [Details] |
| [e.g., Poor transport] | MEDIUM | [Details] |
| [e.g., Unknown developer] | LOW | [Details] |

---

## Sources

| Source | URL | Accessed |
|--------|-----|----------|
| [Source name] | [URL] | [DATE] |
| [Source name] | [URL] | [DATE] |

---

## Recommendation

**Status:** [PROCEED | CAUTION | EXCLUDE]

### Summary

[2-3 sentence summary of location viability]

### If PROCEED

Next step: `/finyx:analyze [location]`

Key considerations for analysis:
- [Consideration 1]
- [Consideration 2]

### If CAUTION

Issues to resolve before proceeding:
- [ ] [Issue 1]
- [ ] [Issue 2]

### If EXCLUDE

**Reason:** [Clear explanation]
**Alternative:** [Suggest alternative if applicable]
