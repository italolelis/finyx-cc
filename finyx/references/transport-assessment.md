# Public Transport Assessment Guide

## Overview

Transport quality directly impacts:
- Parking necessity for rental units
- Tenant pool size
- Resale value
- Rental speed

**Purpose:** Determine if parking is ESSENTIAL or OPTIONAL for each location.

---

## Assessment Criteria

### Network Quality Rating

| Rating | Criteria |
|--------|----------|
| **EXCELLENT** | Multiple modes (U-Bahn/S-Bahn + Tram + Bus), < 5 min walk, < 15 min to center |
| **GOOD** | At least 2 modes, < 10 min walk, < 25 min to center |
| **MODERATE** | Bus service, < 15 min walk, < 40 min to center |
| **POOR** | Limited service, > 15 min walk, > 40 min to center |

### Distance to Stop

| Distance | Assessment |
|----------|------------|
| < 300m (3-4 min walk) | Excellent |
| 300-500m (5-7 min walk) | Good |
| 500-800m (8-12 min walk) | Acceptable |
| > 800m (> 12 min walk) | Poor |

### Service Frequency

| Frequency | Assessment |
|-----------|------------|
| < 5 min peak | Excellent |
| 5-10 min peak | Good |
| 10-20 min peak | Moderate |
| > 20 min peak | Poor |

---

## Research Process

### Step 1: Identify Transport Options

Web searches:
- `"[location] ÖPNV"` (public transport)
- `"[location] Straßenbahn"` (tram)
- `"[location] S-Bahn U-Bahn"`
- `"[location] bus routes"`
- `"[development name] Verkehrsanbindung"` (transport connection)

### Step 2: Map Nearest Stops

Find:
- Nearest stop name
- Distance from development
- Lines serving the stop
- Directions covered

### Step 3: Check Journey Times

Use:
- Google Maps transit directions
- Local transport authority websites
- `"[location] to [city center] public transport"`

### Step 4: Assess Service Quality

Check:
- Operating hours (first/last service)
- Weekend service
- Night service availability
- Planned improvements/extensions

---

## Development Concept Factors

### Car-Reduced Developments

Look for these indicators in marketing/documents:

| Term | Meaning |
|------|---------|
| Autofrei | Car-free |
| Autoarm | Car-reduced |
| Verkehrsberuhigt | Traffic-calmed |
| Quartiersgarage | Shared parking garage |
| Mobilitätskonzept | Mobility concept |
| Carsharing-Stellplätze | Car-sharing spots |

**If present:** Development designed for low car dependency.

### Location Type

| Type | Typical Parking Need |
|------|---------------------|
| City center | Low |
| Inner city | Low-Medium |
| Urban district | Medium |
| Suburban | High |
| Rural | Essential |

---

## Parking Necessity Matrix

| Transport Quality | Development Concept | Parking Verdict |
|-------------------|--------------------|-----------------|
| EXCELLENT | Car-free/reduced | **OPTIONAL** — Non-parking units viable |
| EXCELLENT | Standard | **PREFERRED** — Adds value but not essential |
| GOOD | Car-free/reduced | **PREFERRED** — Consider transport quality |
| GOOD | Standard | **RECOMMENDED** — Most tenants will want |
| MODERATE | Any | **IMPORTANT** — Prioritize parking units |
| POOR | Any | **ESSENTIAL** — Only recommend with parking |

---

## Output Format

Document in location research:

```markdown
## Public Transport

| Item | Details |
|------|---------|
| Nearest Stop | [STOP_NAME] |
| Distance | [X] meters ([Y] min walk) |
| Lines | [LIST: Tram 1, Bus 42, S-Bahn S1] |
| To City Center | [X] minutes |
| Frequency (Peak) | Every [X] minutes |
| Network Quality | [EXCELLENT/GOOD/MODERATE/POOR] |

**Assessment:** [Detailed assessment text]

## Parking Necessity

**Development Concept:** [Standard / Car-reduced / Car-free]

**Verdict:** [ESSENTIAL / IMPORTANT / PREFERRED / OPTIONAL]

**Reasoning:**
[Explanation based on transport + development factors]

**Recommendation:**
- [If ESSENTIAL]: Only consider units with parking
- [If OPTIONAL]: Non-parking units acceptable if yield is better
```

---

## Special Considerations

### Planned Transport Improvements

Check for:
- New tram/metro lines under construction
- Station renovations
- Service frequency improvements

**Note:** Future improvements may increase property value but shouldn't override current assessment for rental purposes.

### Tenant Demographics

| Target Tenant | Parking Importance |
|---------------|-------------------|
| Young professionals | Lower |
| Students | Lowest |
| Families with children | Higher |
| Elderly | Higher |
| Commuters | Depends on workplace |

### Seasonal Factors

Some locations have:
- Tourist season traffic
- University term patterns
- Seasonal employment

Consider how these affect parking demand year-round.

---

## Quick Reference

**Parking OPTIONAL if ALL true:**
- Transport rated EXCELLENT or GOOD
- Development has car-reduced concept
- Urban/city center location
- Target demographic is young professionals/students

**Parking ESSENTIAL if ANY true:**
- Transport rated MODERATE or POOR
- Suburban or rural location
- Family-oriented development
- Limited public transport hours
