# Phase 20: Portfolio Analysis - Research

**Researched:** 2026-04-12
**Domain:** Claude Code slash-command Markdown architecture — insurance portfolio sub-skill + insights integration
**Confidence:** HIGH

## Summary

Phase 20 wires together assets built in Phase 19. The portfolio agent (`finyx-insurance-portfolio-agent.md`) already exists and produces a complete `<portfolio_analysis>` XML block covering adequacy, gaps, overlaps, and cost summary. Phase 20's job is to: (1) create the `portfolio.md` sub-skill that invokes it, (2) add portfolio keywords to the router SKILL.md keyword map, and (3) update `/finyx:insights` to consume `insurance.policies[]` for the allocation Insurance row and gap recommendations.

The insights skill currently reads `insurance.type` and `insurance.monthly_cost` — two old flat fields that Phase 19 replaced with `insurance.policies[]`. Phase 20 must update the insights allocation logic to sum `policies[].premium_monthly` across all policies and populate the Insurance row from that computed total. No schema changes are needed; only prompt logic updates.

**Primary recommendation:** Build the portfolio sub-skill by cloning the structural pattern from `health.md`, remove all health-specific content, add a no-policies interactive entry flow using `AskUserQuestion`, then update SKILL.md router keyword map and insights SKILL.md insurance integration in two targeted edits.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Portfolio Output Structure**
- Portfolio overview organized by tier classification (Mandatory > Essential > Recommended > Situational) matching reference doc tiers
- Gap detection automatically considers family situation from profile.json (family_status, children)
- Overlap detection presented as warning banners per overlap with affected policies named inline
- Adequacy benchmarks show threshold source (e.g., "Hausrat: €650/m² per GDV recommendation")

**Insights Integration**
- Insurance appears as a new "Insurance" row in /finyx:insights allocation table with monthly total + % of net income
- Insights shows insurance gaps as action items in recommendations section ("Missing: Haftpflicht, Rechtsschutz")
- Portfolio sub-skill triggered by "portfolio"/"overview"/"summary" keyword in /finyx:insurance args

**Sub-skill Architecture**
- Portfolio sub-skill spawns the portfolio agent via Task tool (consistent with health sub-skill pattern)
- When no policies exist in profile, prompt user to enter policies interactively via AskUserQuestion before analysis
- Output is conversational only — no file writes (matches health sub-skill pattern)

### Claude's Discretion
- Exact table formatting and column choices for portfolio overview
- How to present the total monthly cost summary
- Specific wording of gap/overlap/adequacy warnings
- How interactive policy entry collects data (number of AskUserQuestion rounds)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| PORT-01 | User can see insurance portfolio overview (all policies, total monthly cost, tier classification) | Portfolio agent Phase 5 output_format already produces this; sub-skill just needs to invoke it and render |
| PORT-02 | User can see coverage gap detection (missing mandatory/essential types based on life situation) | Portfolio agent Phase 3 already produces gap table with priority using identity.family_status and identity.children |
| PORT-03 | User can see overlap/redundancy detection (duplicate coverage across policies) | Portfolio agent Phase 4 already encodes 3 overlap pairs + ADAC note |
| PORT-04 | User can see coverage adequacy check per type (Hausrat ≥€650/m², Haftpflicht ≥€5M, etc.) | Portfolio agent Phase 2 already handles sum_based vs service_based logic with benchmark from reference docs |
| OPT-04 | Insurance costs feed into /finyx:insights financial health report | Insights SKILL.md Phase 1 "Optional fields" section reads insurance.type/monthly_cost — needs update to sum policies[].premium_monthly |
</phase_requirements>

## Standard Stack

### Core
| File | Pattern | Purpose |
|------|---------|---------|
| `skills/insurance/sub-skills/portfolio.md` | Plain Markdown, no frontmatter | Sub-skill dispatched by router; invokes portfolio agent via Task |
| `skills/insurance/SKILL.md` | Router keyword map edit | Add "portfolio", "overview", "summary" → sub_skill_type = "portfolio" |
| `skills/insights/SKILL.md` | Insurance integration edit | Update Phase 1 optional fields + Phase 3 allocation agent prompt to sum policies[] |

### Supporting
| File | Role | Status |
|------|------|--------|
| `skills/insurance/agents/finyx-insurance-portfolio-agent.md` | Does all analysis work; returns `<portfolio_analysis>` | Exists (Phase 19) |
| `skills/insurance/references/germany/*.md` | 11 reference docs loaded by agent in execution_context | Exists (Phase 19) |
| `skills/profile/references/profile.json` | `insurance.policies[]` schema | Exists (Phase 19) |
| `.finyx/profile.json` | Runtime data source | Written by /finyx:profile |

### Alternatives Considered
| Standard Approach | Alternative | Why Standard Wins |
|---|---|---|
| Single AskUserQuestion round for empty-portfolio policy entry | Multiple sequential question rounds | Single round matches Phase 0 pattern in health.md; keeps interaction surface minimal |
| Sum policies[].premium_monthly in SKILL.md prompt logic | Agent-computed total | SKILL.md already reads profile.json in execution_context; sum is trivial inline logic, no extra agent spawn needed |

## Architecture Patterns

### Established Sub-skill Pattern (from health.md)

```
# [Type] Insurance Sub-skill

<!-- Sub-skill loaded by router SKILL.md. All CLAUDE_SKILL_DIR paths resolve to skills/insurance/ -->

<objective>...</objective>

<process>
## Phase 0: [Pre-flight / Data check]
## Phase 1: [Validation / Profile read]
## Phase N: [Agent spawning via Task]
## Phase N+1: [Output synthesis]
</process>

<error_handling>...</error_handling>

<notes>...</notes>
```

Sub-skill contract rules (from SKILL.md notes section):
- No YAML frontmatter
- No `<execution_context>` block (router already loads disclaimer.md and profile.json)
- Use `${CLAUDE_SKILL_DIR}` for all path references
- Start with `# [Type] Insurance Sub-skill` heading

### Portfolio Sub-skill Process Flow

```
Phase 0: Profile check — does insurance.policies[] exist and have entries?
  → If empty/missing: AskUserQuestion to collect policy data interactively
Phase 1: Disclaimer emission (match health.md disclaimer-first pattern)
Phase 2: Spawn portfolio agent via Task tool
Phase 3: Render <portfolio_analysis> output as conversational advisory text
          - Tier-ordered overview (Mandatory > Essential > Recommended > Situational)
          - Adequacy table with benchmark source attribution
          - Gap table with priority + family-context reason
          - Overlap warning banners (named policies inline)
          - Cost summary with monthly total
```

### Router Keyword Map Edit (SKILL.md)

Add to Phase 0 keyword map:
```
- "portfolio", "overview", "summary", "all policies", "alle versicherungen", "übersicht" → sub-skill: portfolio
```

Add to Phase 0 singleSelect fallback menu:
```
- Portfolio overview (all policies, gaps, overlaps, adequacy summary)
```

Add to Phase 1 mapping:
```
- "Portfolio overview (all policies, gaps, overlaps, adequacy summary)" → sub_skill_type = "portfolio"
```

Add to `<error_handling>` unknown type list:
```
  - portfolio        (Portfolio — all policies, gaps, overlaps, cost summary)
```

Add to SKILL.md notes section — Sub-skill Tool Permissions:
```
- portfolio.md: Read, Task, AskUserQuestion
```

### Insights Integration Pattern

**Current state in insights SKILL.md Phase 1 (Optional fields):**
```
- `insurance.type` and `insurance.monthly_cost` — if populated, insurance costs are included
  in allocation analysis as a "needs" line item. If absent or null: skip silently.
```

**Phase 19 added `insurance.policies[]`; the flat `insurance.type` / `insurance.monthly_cost` fields no longer exist in the schema.** The insights integration is broken against the new schema.

**Required update — Phase 1 optional fields block:**
```
- `insurance.policies[]` — if array is non-empty, compute total monthly premium by summing
  `policies[].premium_monthly` across all entries. Include as "Insurance" needs line item
  with label "Insurance (X policies)" and monthly total. If array is empty or absent: skip silently.
  Gap types (from portfolio analysis, if run) are surfaced as action items only when
  portfolio sub-skill has been run in the same session — otherwise omit gap row.
```

**Required update — Phase 3 Allocation Agent Task prompt:**
```
If `.finyx/profile.json` contains a non-empty `insurance.policies[]` array:
- Sum all `policies[].premium_monthly` values to compute total monthly insurance cost
- Include as a "needs" category line item labeled "Insurance ([N] policies — €[total]/month)"
- If any policy has premium_monthly of 0 or null, note as [DATA GAP] for that policy
- If insurance.policies[] is absent or empty: skip — do not include any insurance line
```

**Required update — Phase 5 Top-5 Recommendations (cross-advisor patterns section):**

CAL-05 currently checks `insurance.type == "PKV"` and `insurance.monthly_cost`. Update trigger to:
```
- A `health` policy in `insurance.policies[]` has premium_monthly > 400 AND savings rate is below 20%
```

### Interactive Policy Entry (Empty Portfolio Flow)

When `insurance.policies[]` is empty or absent, the portfolio sub-skill must collect basic policy data before spawning the agent. The locked decision is to use AskUserQuestion before analysis.

Recommended approach (within discretion scope):
- Single AskUserQuestion with multiSelect listing the 11 known insurance types: "Which insurance types do you currently have? Select all that apply."
- For each selected type: a second round collecting provider name and monthly premium (can be one AskUserQuestion per type or batched — this is discretionary)
- Write collected entries to `.finyx/profile.json` via Write tool before spawning portfolio agent

**Note:** This requires Write in the sub-skill's tool usage. The router SKILL.md already declares Write in `allowed-tools` (it was included for health.md needs). No frontmatter change needed.

### Anti-Patterns to Avoid

- **Do not add `<execution_context>` to portfolio.md** — the router already loads disclaimer.md and profile.json. Adding it again creates duplicate context loading.
- **Do not hardcode benchmark thresholds in portfolio.md** — benchmarks must come from reference docs loaded by the portfolio agent. The sub-skill only orchestrates; the agent does analysis.
- **Do not pass `property_size_sqm` to portfolio agent without first checking profile** — the Hausrat adequacy check needs it; if absent, the agent marks UNKNOWN. Sub-skill should check profile for `investor.livingAreaSqm` or similar and pass it in the Task prompt if found.
- **Do not update the old flat `insurance.type` / `insurance.monthly_cost` pattern** — these fields are gone in the Phase 19 schema. Insights integration must target `insurance.policies[]` exclusively.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead |
|---------|-------------|-------------|
| Portfolio analysis logic | Custom gap/overlap/adequacy logic in sub-skill | Portfolio agent already encodes all logic — sub-skill only spawns it |
| Benchmark threshold values | Hardcode in sub-skill | Agent loads reference docs and reads benchmarks at runtime |
| Monthly premium total | Manual sum in agent prompt | Simple inline sum instruction in Task prompt; agent handles the arithmetic |
| Tier classification | Define tiers in sub-skill | Reference docs and agent already define Mandatory/Essential/Recommended/Situational |

## Common Pitfalls

### Pitfall 1: Insights Old Insurance Field References
**What goes wrong:** Insights SKILL.md Phase 1 and Phase 3 allocation agent prompt reference `insurance.type` and `insurance.monthly_cost` — fields that no longer exist after Phase 19 extended the schema to `insurance.policies[]`.
**Why it happens:** Phase 19 changed the schema but insights SKILL.md was not updated in that phase.
**How to avoid:** Update both locations in insights SKILL.md: (1) Phase 1 optional fields description, (2) Phase 3 allocation agent Task prompt template.
**Warning signs:** Insights runs silently without an Insurance row even when the user has policies entered.

### Pitfall 2: Sub-skill Writing to profile.json Without Phase 19 Schema Compliance
**What goes wrong:** Interactive policy entry in the empty-portfolio flow writes policy objects that don't match the `insurance.policies[]` schema (missing required fields like `id`, `start_date`, `kuendigungsfrist_months`).
**Why it happens:** Manual Write instructions in a sub-skill may not enforce all schema fields.
**How to avoid:** The interactive entry Task prompt must explicitly enumerate all required policy fields from the profile.json schema. Use null for optional fields the user hasn't provided. Reference the `_insurance_schema` annotation in profile.json.

### Pitfall 3: Portfolio Sub-skill Missing property_size_sqm for Hausrat Adequacy
**What goes wrong:** Portfolio agent Phase 2 notes that Hausrat adequacy "cannot be assessed without Wohnfläche" and marks as UNKNOWN even when the user has Hausrat coverage and a known apartment size.
**Why it happens:** The Task prompt doesn't pass `property_size_sqm` to the agent.
**How to avoid:** In the portfolio sub-skill Phase 2 Task prompt construction, check profile for any field that stores living area (e.g., search `profile.json` for a sqm field). If found, include it as `property_size_sqm` in the Task prompt. If not found, note to user that Hausrat adequacy requires Wohnfläche input via /finyx:profile.

### Pitfall 4: Router Menu Missing Portfolio Entry
**What goes wrong:** User types `/finyx:insurance` with no args, gets the singleSelect menu, but "Portfolio overview" is not in the list — so portfolio is only reachable by keyword.
**Why it happens:** Router SKILL.md Phase 0 singleSelect list is edited independently from the keyword map.
**How to avoid:** Edit both simultaneously — keyword map AND singleSelect menu AND answer mapping all need the portfolio entry added in the same edit.

### Pitfall 5: Insights Gap Recommendations Without Portfolio Run
**What goes wrong:** Insights shows "Missing: Haftpflicht, Rechtsschutz" as action items even though the portfolio agent was never run in this session — the data comes from the policies[] array directly, not from a portfolio analysis.
**Why it happens:** The locked decision says insights shows gap action items, but the portfolio agent determines gaps by running full analysis.
**How to avoid:** Insights gap detection for the recommendations section should derive from the policies[] array directly (check which of the 11 known types are absent), not require a prior portfolio agent run. This is a simple set-difference check the insights allocation agent or the orchestrator can do inline without spawning the portfolio agent.

## Code Examples

### Task Prompt Pattern for Portfolio Agent (from sub-skill)

```markdown
Spawn `finyx-insurance-portfolio-agent` with this prompt:

You are the finyx-insurance-portfolio-agent. Analyze the user's full insurance portfolio.

Profile data is at `.finyx/profile.json`. Reference docs for all 11 insurance types are already
loaded in your execution_context.

[If property_size_sqm is known from profile:]
property_size_sqm: [value]

[If property_size_sqm is unknown:]
property_size_sqm: null — Hausrat adequacy will be marked UNKNOWN; advise user to add
living area to profile via /finyx:profile.

Complete all phases of your process and return your output wrapped in <portfolio_analysis> tags.
```

### Insights Policies Sum Logic (inline in Task prompt)

```markdown
If `.finyx/profile.json` contains a non-empty `insurance.policies[]` array:
- Sum all `policies[].premium_monthly` values: total_monthly_insurance = Σ policies[i].premium_monthly
- Include as a "needs" category line item labeled "Insurance ([N] policies — €[total]/month)"
  where N = count of policies and total = computed sum
- If any policy has premium_monthly of 0 or null, note "[DATA GAP: premium_monthly missing for
  [policy.id]]" inline
- Do NOT reference insurance.type or insurance.monthly_cost — those fields do not exist in the schema
- If insurance.policies[] is absent or empty: skip — do not include any insurance line
```

### Empty Portfolio Interactive Entry (AskUserQuestion pattern)

```markdown
## Phase 0: Policy Data Check

Read `.finyx/profile.json` (already loaded by router).
Check `insurance.policies[]`.

If array is empty or key is absent:
Use AskUserQuestion with multiSelect:
"You have no insurance policies recorded. Which insurance types do you currently have?
Select all that apply, then we'll enter the details for each."

Options:
- Health insurance (GKV or PKV)
- Personal liability (Haftpflichtversicherung)
- Household contents (Hausratversicherung)
- Car insurance (Kfz-Versicherung)
- Legal protection (Rechtsschutzversicherung)
- Dental supplement (Zahnzusatzversicherung)
- Term life (Risikolebensversicherung)
- Travel insurance (Reiseversicherung)
- Bicycle insurance (Fahrradversicherung)
- Roadside assistance (Kfz-Schutzbriefversicherung)
- Rental deposit insurance (Mietkautionsversicherung)
- None — I want to see what I'm missing (gap analysis only)
```

## State of the Art

| Old Pattern | Current Pattern | Impact |
|-------------|-----------------|--------|
| `insurance.type` + `insurance.monthly_cost` flat fields | `insurance.policies[]` array (Phase 19) | Insights SKILL.md must be updated; old field references are dead |
| Gap detection with hardcoded type lists | Agent loads reference docs, reads who-needs-it from Overview section | Ensures gap priority reasons cite authoritative source |

## Environment Availability

Step 2.6: SKIPPED (no external dependencies — all work is Markdown prompt file edits)

## Open Questions

1. **Profile field for living area / Wohnfläche**
   - What we know: Portfolio agent needs `property_size_sqm` for Hausrat adequacy. Health sub-skill does not pass this.
   - What's unclear: Whether profile.json currently has a field for living area (it does not appear in the template — `criteria.minSize`/`maxSize` exist for investment property criteria, not for user's own residence).
   - Recommendation: Sub-skill should check `criteria.minSize` as a proxy (if set) or ask user inline. Document as a known data gap if neither is available.

2. **Insights gap action items derivation**
   - What we know: Locked decision says insights shows "Missing: Haftpflicht, Rechtsschutz" in recommendations.
   - What's unclear: Whether the planner should derive this from policies[] set-difference in the SKILL.md orchestrator, or pass it to the allocation agent to derive, or require a prior portfolio agent run.
   - Recommendation: Implement as a simple set-difference check in the allocation agent Phase 1 (it already reads profile.json): compare policies[].type values against the ESSENTIAL type list. No portfolio agent spawn needed for this.

## Sources

### Primary (HIGH confidence)
- Direct file reads: `skills/insurance/SKILL.md`, `skills/insurance/agents/finyx-insurance-portfolio-agent.md`, `skills/insurance/sub-skills/health.md`, `skills/insights/SKILL.md`, `skills/insights/agents/finyx-allocation-agent.md`, `skills/profile/references/profile.json`
- All findings derived from actual source files — no external sources needed for this architecture-internal phase

### Secondary (MEDIUM confidence)
- None required

### Tertiary (LOW confidence)
- None

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all files exist and were read directly
- Architecture: HIGH — patterns established by health.md and insights.md are fully documented
- Pitfalls: HIGH — schema mismatch (old flat fields vs policies[]) confirmed by direct comparison of insights SKILL.md against profile.json schema

**Research date:** 2026-04-12
**Valid until:** Stable until Phase 21 adds per-type sub-skills (no impact on portfolio sub-skill)
