# Phase 21: Per-Type Sub-skills (Tier 1-2) - Research

**Researched:** 2026-04-12
**Domain:** Claude Code slash-command sub-skill authoring — German insurance domain
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- Each sub-skill follows the health sub-skill's multi-phase pattern: preferences > validation > agent spawn > disclaimer > synthesis > recommendation
- "Current coverage vs benchmark" presented as side-by-side table: Your Coverage vs Recommended Minimum, with pass/fail per criterion
- Market comparison spawns the research agent via Task tool (consistent with health pattern, enables live WebSearch)
- Sub-skills are read-only advisory — no file writes (consistent with health sub-skill pattern)
- Kfz: 3 coverage levels presented as tabbed comparison — Haftpflicht vs Teilkasko vs Vollkasko with cost delta and when-to-upgrade guidance
- SF-Klasse (no-claims bonus) collected via AskUserQuestion (0-35 range) with explanation
- Kfz sub-skill has extended Phase with vehicle-specific questions (car age, annual km, garage) before agent spawn
- Cancellation deadline data read from `profile.insurance.policies[]` fields: `kuendigungsfrist_months`, `vertrag_beginn`, `sonderkundigungsrecht`
- Sonderkuendigungsrecht windows presented as alert banner when open or within 30 days
- When cancellation fields are missing: show "Unknown — add policy details via /finyx:insurance portfolio" with link

### Claude's Discretion

- Exact question sets for each sub-skill's preferences phase (varies by type complexity)
- How many research agent search queries per type
- Specific wording of benchmark comparison tables
- How to present the recommendation (prose vs structured)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| TYPE-01 | Privathaftpflicht (personal liability) sub-skill | haftpflicht.md reference doc fully specifies benchmarks; health.md pattern is the template |
| TYPE-02 | Hausratversicherung (household contents) sub-skill | hausrat.md has €650/m² benchmark and Unterversicherungsverzicht logic; requires living area from profile |
| TYPE-03 | Kfz-Versicherung (car insurance — 3 tiers) sub-skill | kfz.md defines all 3 tiers, SF-Klasse, Typklasse, Regionalklasse; most complex sub-skill |
| TYPE-04 | Rechtsschutzversicherung (legal protection) sub-skill | rechtsschutz.md defines modules (Berufs-, Verkehrs-, Miet-, Privatrechtsschutz) and 3-month Wartezeit |
| TYPE-05 | Zahnzusatzversicherung (dental supplement) sub-skill | zahnzusatz.md defines Staffelung pattern, GKV Festzuschuss context, Wartezeit logic |
| TYPE-06 | Risikolebensversicherung (term life) sub-skill | risikoleben.md defines benchmark formula (3-5x income or mortgage balance), health underwriting note, fixed-term renewal behavior |
| OPT-01 | Market comparison per type via WebSearch (criteria-based) | Research agent already parameterized — sub-skills pass `insurance_type` and optional params; §34d GewO compliance already encoded in agent |
| OPT-03 | Cancellation deadline tracking (Kündigungsfrist + Sonderkündigungsrecht) | All 6 reference docs contain Cancellation Rules sections; profile schema already has `kuendigungsfrist_months`, `sonderkundigungsrecht`, `start_date`, `renewal_date` fields |
</phase_requirements>

---

## Summary

Phase 21 creates 6 plain-Markdown sub-skill files under `skills/insurance/sub-skills/`. All infrastructure is complete: the router SKILL.md already dispatches to these filenames, the research agent accepts any `insurance_type` slug, and all 6 reference docs are present with benchmarks, keyword maps, and cancellation rules.

The pattern is well-established from `health.md` (~600 lines) and `portfolio.md` (~274 lines). The 5 simpler types (haftpflicht, hausrat, rechtsschutz, zahnzusatz, risikoleben) each follow a streamlined 4-phase structure: preferences → validation/profile read → disclaimer → research agent spawn → coverage benchmark table → cancellation deadline → recommendation. Kfz is the exception: it has an extended preferences phase collecting SF-Klasse, vehicle age, annual km, and garage status before spawning the research agent, plus a 3-tier coverage comparison table.

Cancellation tracking (OPT-03) is implemented inline in every sub-skill by reading `insurance.policies[]` entries matching the type slug, computing deadline proximity from `start_date + kuendigungsfrist_months`, and surfacing an alert banner when the window is open or within 30 days. Market comparison (OPT-01) is fulfilled by spawning the existing research agent — no new agent work required.

**Primary recommendation:** Model each non-Kfz sub-skill as a ~150-200 line document following health.md's phase structure but stripped of health-specific complexity (no JAEG gate, no parallel agent spawn, no health flags). Kfz gets its own extended template (~250-300 lines) matching CONTEXT.md's locked Kfz decisions.

---

## Standard Stack

This project has no library dependencies. The "stack" is file conventions.

### Core Conventions (HIGH confidence — verified from existing sub-skills)

| Asset | Location Pattern | Notes |
|-------|-----------------|-------|
| Sub-skill file | `skills/insurance/sub-skills/{type}.md` | Plain Markdown, no YAML frontmatter |
| Reference doc | `${CLAUDE_SKILL_DIR}/references/germany/{type}.md` | Loaded inline by sub-skill Read calls |
| Research agent | `skills/insurance/agents/finyx-insurance-research-agent.md` | Spawned via Task; accepts `insurance_type` param |
| Profile | `.finyx/profile.json` | Loaded by router's execution_context; available at sub-skill runtime |
| Disclaimer | `${CLAUDE_SKILL_DIR}/references/disclaimer.md` | Loaded by router; sub-skill emits it in Phase 1 |

### Router Integration (already complete — no changes needed)

The router SKILL.md dispatches to exactly these filenames for Phase 21:
- `sub-skills/haftpflicht.md`
- `sub-skills/hausrat.md`
- `sub-skills/kfz.md`
- `sub-skills/rechtsschutz.md`
- `sub-skills/zahnzusatz.md`
- `sub-skills/risikoleben.md`

The router's `allowed-tools` already lists all needed tools. No SKILL.md changes required.

---

## Architecture Patterns

### Sub-skill File Contract (HIGH confidence — from health.md and portfolio.md)

```
# {Type} Insurance Sub-skill

<!-- Sub-skill loaded by router SKILL.md. All CLAUDE_SKILL_DIR paths resolve to skills/insurance/ -->

<objective>
...
</objective>

<process>

## Phase 0: Preferences
## Phase 1: Validation and Profile Read
## Phase 2: Disclaimer
## Phase 3: Coverage Benchmark Comparison
## Phase 4: Cancellation Deadline Check
## Phase 5: Research Agent Spawn
## Phase 6: Recommendation

</process>

<error_handling>
...
</error_handling>

<notes>
...
</notes>
```

Rules (from SKILL.md sub-skill file contract):
- No YAML frontmatter (sub-skills are not registered commands)
- No `<execution_context>` block (router already loads disclaimer.md and profile.json)
- Use `${CLAUDE_SKILL_DIR}` for all path references
- Start with `# [Type] Insurance Sub-skill` heading

### Pattern 1: Standard Non-Kfz Sub-skill Phase Structure

**What:** 6-phase linear flow for simpler types
**When to use:** haftpflicht, hausrat, rechtsschutz, zahnzusatz, risikoleben

**Phase 0 — Preferences:** AskUserQuestion to collect 2-3 type-specific questions (coverage priorities, budget range, current situation). See per-type details below.

**Phase 1 — Validation and Profile Read:** Check profile exists. Read `insurance.policies[]` to find matching policy entry by `type` slug. Extract `coverage_amount`, `premium_monthly`, `kuendigungsfrist_months`, `sonderkundigungsrecht`, `start_date`, `renewal_date`. Set `existing_policy_found = true/false`.

**Phase 2 — Disclaimer:** Emit header banner + full disclaimer.md content + insurance-specific addendum. Pattern identical to portfolio.md Phase 1.

**Phase 3 — Coverage Benchmark Comparison:** Read reference doc benchmarks. If `existing_policy_found`, build side-by-side table: "Your Coverage vs Recommended Minimum" with PASS/FAIL per criterion. If no existing policy: show benchmarks only with "Add your policy via `/finyx:insurance portfolio`" note.

**Phase 4 — Cancellation Deadline Check:** Compute deadline from profile data. See Cancellation Tracking pattern below.

**Phase 5 — Research Agent Spawn:** Spawn `finyx-insurance-research-agent` via Task tool. Pass `insurance_type` slug and optional params. Render `<insurance_research_result>` output.

**Phase 6 — Recommendation:** Synthesize coverage gap + market context into 2-3 paragraph advisory. Include concrete next steps.

### Pattern 2: Kfz Extended Sub-skill Phase Structure

**What:** Extended phase structure for Kfz — vehicle-specific data collection + 3-tier comparison
**When to use:** kfz sub-skill only (per CONTEXT.md locked decisions)

Extended preferences phase (Phase 0):
```
AskUserQuestion questions:
1. Current coverage tier (singleSelect): Haftpflicht only / Haftpflicht + Teilkasko / Vollkasko
2. SF-Klasse (0–35 range) with explanation: "Your no-claims bonus class — 0 = new driver, 35 = 35 claim-free years"
3. Vehicle age in years (numeric input)
4. Annual km driven (singleSelect): <10,000 / 10,000–20,000 / 20,000–30,000 / 30,000+
5. Garage available? (boolean): yes / no
6. Vehicle financed? (boolean): yes / no
```

Coverage benchmark table (Phase 3) — 3-column comparison:
```
| Criterion          | Haftpflicht   | Teilkasko     | Vollkasko     |
|--------------------|---------------|---------------|---------------|
| Third-party damage | MANDATORY     | Included      | Included      |
| Own theft          | Not covered   | COVERED       | COVERED       |
| Own accident       | Not covered   | Not covered   | COVERED       |
| Vandalism          | Not covered   | Not covered   | COVERED       |
| Typical premium    | €X–Y/mo       | €X–Y/mo       | €X–Y/mo       |
```

Plus: "When to upgrade" guidance based on vehicle age from kfz.md Coverage Benchmarks:
- 0-3 years → Vollkasko recommended
- 3-7 years → Vollkasko or Teilkasko
- >7 years → Teilkasko or Haftpflicht only
- Financed → Vollkasko required

Research agent spawn passes: `insurance_type: kfz`, `user_city` (from profile if available), `current_premium_monthly` (from matching policy if found).

### Pattern 3: Cancellation Tracking (OPT-03)

**What:** Deadline computation from profile data, surfaced as alert banner
**When to use:** All 6 sub-skills, Phase 4

```
From profile insurance.policies[], find entry where type == "{slug}":
  - kuendigungsfrist_months → N
  - start_date → parse to compute anniversary (Hauptfalligkeit)
  - renewal_date → use directly if present (overrides computed anniversary)
  - sonderkundigungsrecht → boolean

Compute Hauptfalligkeit:
  - If renewal_date present: use it
  - Else: compute next anniversary from start_date

Compute deadline:
  deadline = Hauptfalligkeit - kuendigungsfrist_months months

Compute days_until_deadline = deadline - today

Cases:
  1. days_until_deadline < 0: deadline has passed — next opportunity is next year
  2. 0 <= days_until_deadline <= 30: ALERT banner — "X days to cancel"
  3. days_until_deadline > 30: show deadline date informatively

Sonderkuendigungsrecht check:
  - If sonderkundigungsrecht == true: show alert banner "Sonderkündigungsrecht currently open"
```

When profile fields are missing (kuendigungsfrist_months or start_date absent):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 CANCELLATION DEADLINE: UNKNOWN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Policy details not found. Add start date and cancellation
period via `/finyx:insurance portfolio` to track deadlines.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Special case — Risikoleben:** `renewal_date` is the policy END DATE (Vertragsende), not an annual renewal. The cancellation display must say "Policy expires: {date}" not "Next renewal." The Kundigungsfrist applies to mid-term cancellation only (no cash value, no refund).

### Anti-Patterns to Avoid

- **Naming specific providers or tariffs (non-health types):** All 5 non-health, non-Kfz types and Kfz must receive criteria-only output from research agent. §34d GewO compliance is enforced by the research agent — sub-skills must not override this by adding provider names to Task prompts.
- **Hardcoding benchmark thresholds:** Read from reference docs at runtime. The research agent has an anti-hallucination rule against hardcoded benchmarks — sub-skills should mirror this for the benchmark comparison table.
- **Writing to profile.json:** These sub-skills are read-only. The portfolio sub-skill handles profile writes. Adding write calls here breaks the pattern.
- **Parallel agent spawning:** Unlike health.md (two parallel agents), these sub-skills spawn only one agent (research agent). Do not add a calc agent — there is no calc agent for non-health types.
- **Placing <execution_context> in sub-skill files:** The router already loads disclaimer.md and profile.json. Sub-skills must NOT duplicate this — they'd cause double loading.

---

## Per-Type Domain Details

### Privathaftpflicht (haftpflicht)

**Reference doc:** `skills/insurance/references/germany/haftpflicht.md`
**Profile type slug:** `haftpflicht`
**Key benchmark:** Deckungssumme ≥ €5,000,000 (pauschal covering Personen-, Sach-, Vermogensschaden)
**Coverage comparison columns:** Deckungssumme, Mietsachschaden (included?), Schlusselverlust (included?), Gefalligkeitsschaden (included?), Forderungsausfalldeckung (included?), Selbstbeteiligung
**Preferences to collect:** Budget range, whether renting (Mietsachschaden relevance), pets (Hundehaftpflicht overlap check)
**Cancellation:** 3 months before Hauptfalligkeit; Sonderkündigungsrecht on premium increase or claim settlement
**Complexity:** LOW — simplest sub-skill; good first implementation target

### Hausratversicherung (hausrat)

**Reference doc:** `skills/insurance/references/germany/hausrat.md`
**Profile type slug:** `hausrat`
**Key benchmark:** Versicherungssumme ≥ €650/m² × Wohnflache; Unterversicherungsverzicht clause required
**Coverage comparison columns:** Versicherungssumme vs computed minimum (living_area × 650), Unterversicherungsverzicht (included?), Elementarschaden (included?), Einbruchdiebstahl (included?)
**Profile dependency:** Needs `investor.livingAreaSqm` or equivalent to compute minimum. If absent: show "UNKNOWN — add living area via `/finyx:profile`" (same pattern as portfolio sub-skill)
**Preferences to collect:** Living area if not in profile, Elementarschaden add-on interest, Fahrrad-Zusatz interest
**Cancellation:** 3 months before Hauptfalligkeit; address change (Umzug) triggers Sonderkündigungsrecht if premium increases
**Complexity:** LOW-MEDIUM (living area lookup adds a conditional)

### Kfz-Versicherung (kfz)

**Reference doc:** `skills/insurance/references/germany/kfz.md`
**Profile type slug:** `kfz`
**Key benchmark:** Haftpflicht is MANDATORY (§1 PflVG); Vollkasko strongly recommended for vehicles ≤3 years or financed
**Tier comparison table:** 3 columns (Haftpflicht / Teilkasko / Vollkasko), rows: third-party damage, own theft, weather damage, own accident, vandalism, typical price range
**SF-Klasse collection:** AskUserQuestion with range 0-35; explain impact (SF 35 = 70-80% discount vs SF 0)
**Vehicle data to collect:** Current tier, SF-Klasse, vehicle age, annual km, garage status, financed status
**Cancellation:** 1 month (Kundigungsfrist by 30.11 for 01.01 renewal); Sonderkündigungsrecht after claim or SF-Klasse change
**Profile note:** `coverage_components` field contains current tier(s) — read this to show "Current tier: Vollkasko"
**Complexity:** HIGH — most questions, 3-tier comparison, SF-Klasse impact explanation

### Rechtsschutzversicherung (rechtsschutz)

**Reference doc:** `skills/insurance/references/germany/rechtsschutz.md`
**Profile type slug:** `rechtsschutz`
**Key benchmark:** Module coverage — Berufs-, Verkehrs-, Miet-, Privatrechtsschutz; Wartezeit 3 months
**Coverage comparison columns:** Active modules (from coverage_components), Wartezeit status, Selbstbeteiligung
**Profile dependency:** `coverage_components` contains active modules (e.g., `["Berufsrechtsschutz", "Verkehrsrechtsschutz"]`)
**Preferences to collect:** Employment status (employee = Berufsrechtsschutz priority), renter vs owner (Mietrechtsschutz priority), driver (Verkehrsrechtsschutz priority)
**Domain-specific note:** Rechtsschutz coverage_amount is null (service_based) — never show a monetary sum benchmark; show module coverage instead
**Cancellation:** 3 months before Hauptfalligkeit; ongoing covered cases NOT voided by cancellation
**Complexity:** MEDIUM (module-based coverage requires different comparison logic than sum-based types)

### Zahnzusatzversicherung (zahnzusatz)

**Reference doc:** `skills/insurance/references/germany/zahnzusatz.md`
**Profile type slug:** `zahnzusatz`
**Key benchmark:** Zahnersatz reimbursement ≥70% (minimum), ≥80-100% (recommended); Staffelung pattern is key risk
**Coverage comparison columns:** Zahnersatz % reimbursement, Zahnbehandlung %, PZR included?, Implantate covered?, Wartezeit, Staffelung annual cap (year 1)
**Bonusheft context:** Show GKV Festzuschuss baseline (60-75% depending on Bonusheft status) — this explains why Zahnzusatz matters
**Preferences to collect:** Immediate dental needs (Staffelung risk), implant needs, children (KFO relevance)
**Cancellation:** 3 months; switching warning — new Wartezeit at new policy; mid-treatment switching should be avoided
**Complexity:** MEDIUM (Staffelung explanation adds complexity; GKV baseline context is informative)

### Risikolebensversicherung (risikoleben)

**Reference doc:** `skills/insurance/references/germany/risikoleben.md`
**Profile type slug:** `risikoleben`
**Key benchmark:** Versicherungssumme = max(gross_income × 3-5, outstanding_mortgage_balance)
**Coverage comparison:** Current Versicherungssumme vs computed recommended sum; term adequacy (does term extend to youngest child age 25 or mortgage payoff?)
**Profile dependencies:** `countries.germany.gross_income` for benchmark calculation; check for mortgage balance in profile (investor data)
**Term type note:** Display whether current policy is level or declining term (from coverage_components)
**Health underwriting note:** Include a note that health underwriting applies — recommend not cancelling before new policy is approved
**Cancellation:** 3 months (mid-term); renewal_date = POLICY END DATE (not annual renewal); no Ruckkaufswert on cancellation
**Complexity:** MEDIUM-HIGH (benchmark formula needs profile data; health underwriting warning is important)

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Market comparison with WebSearch | Custom search logic in sub-skill | `finyx-insurance-research-agent` via Task | Agent already handles query construction, §34d GewO compliance, source prioritization, and output formatting |
| Coverage benchmarks | Inline hardcoded thresholds | Read from `${CLAUDE_SKILL_DIR}/references/germany/{type}.md` at runtime | Reference docs are the single source of truth; hardcoding breaks when docs are updated |
| Cancellation deadline math | Complex date calculation | Simple inline arithmetic in sub-skill: `renewal_date - kuendigungsfrist_months` | Dates are already normalized to ISO strings in profile; no library needed |
| Profile field lookup | Custom profile parsing | Direct field access pattern from portfolio.md | Pattern already established and tested |

---

## Common Pitfalls

### Pitfall 1: Forgetting sonderkundigungsrecht Field Name

**What goes wrong:** CONTEXT.md mentions `vertrag_beginn` as a field name, but the actual profile schema (from portfolio.md Phase 0 policy object) uses `start_date`. The field `vertrag_beginn` does not exist in the schema.
**Why it happens:** CONTEXT.md was written from memory; the canonical schema is in portfolio.md Phase 0.
**How to avoid:** Always read from `start_date` (ISO date) and `kuendigungsfrist_months` (number). The `sonderkundigungsrecht` field is a boolean (true/false), not a date.
**Warning signs:** If sub-skill code references `vertrag_beginn`, it will silently get null.

### Pitfall 2: Wrong coverage_amount Interpretation Per Type

**What goes wrong:** Using `coverage_amount` as a monetary limit for service-based types (rechtsschutz).
**Why it happens:** coverage_amount field exists for all types but means different things.
**How to avoid:** Per reference docs:
- haftpflicht: coverage_amount = Deckungssumme (pauschal) in EUR — sum-based
- hausrat: coverage_amount = Versicherungssumme in EUR — sum-based
- kfz: coverage_amount = Deckungssumme Haftpflicht (EUR for Haftpflicht tier); null for Teilkasko/Vollkasko
- rechtsschutz: coverage_amount = null always (service-based — record null)
- zahnzusatz: coverage_amount = year-1 Staffelung cap (EUR); null if unlimited
- risikoleben: coverage_amount = Versicherungssumme (death benefit) in EUR — sum-based

### Pitfall 3: Risikoleben renewal_date Is Not an Annual Renewal

**What goes wrong:** Computing cancellation deadline from renewal_date as if it's an annual date.
**Why it happens:** All other types have annual Hauptfalligkeit; Risikoleben has a fixed end date.
**How to avoid:** Risikoleben's `renewal_date` field stores `Vertragsende / Ablaufdatum` — the fixed policy expiry. Display it as "Policy expires: {date}" not "Next renewal." The Kundigungsfrist logic applies to mid-term cancellation only (no cash value).

### Pitfall 4: Research Agent Output for Kfz Mentions SF-Klasse

**What goes wrong:** Sub-skill collects SF-Klasse from user but never passes it to the research agent — agent then provides generic market context without personalizing to SF-Klasse.
**Why it happens:** The research agent accepts optional params including `current_premium_monthly`. SF-Klasse is a key price factor.
**How to avoid:** Pass collected vehicle data in the Task prompt as additional context: `sf_klasse: {value}`, `vehicle_age_years: {value}`, `annual_km: {value}`. The research agent uses these for Market Context personalization (reference doc Phase 3 mentions "Key price factors").

### Pitfall 5: Sub-skill Tries to Write profile.json

**What goes wrong:** Adding Write calls to update policy details (e.g., after user confirms a cancellation).
**Why it happens:** Portfolio sub-skill does write to profile — copying that pattern blindly.
**How to avoid:** These 6 sub-skills are advisory-only and write NO files. The portfolio sub-skill is the only write pathway. All updates directed to `/finyx:insurance portfolio`.

### Pitfall 6: Hausrat Benchmark Requires Living Area

**What goes wrong:** Computing Versicherungssumme minimum without living area, or crashing when the field is absent.
**Why it happens:** `living_area_sqm` is not always populated in profile.
**How to avoid:** Mirror portfolio.md's property_size_sqm pattern: check `investor.livingAreaSqm`, `criteria.minSize`, or any sqm-related field. If not found: mark adequacy as UNKNOWN and prompt user to add it. Do not skip the phase entirely.

---

## Code Examples

### Coverage Benchmark Comparison Table (Haftpflicht example)

```markdown
## Your Coverage vs Recommended Minimum

| Criterion | Your Coverage | Recommended Minimum | Status |
|-----------|--------------|---------------------|--------|
| Deckungssumme | €{coverage_amount} | ≥€5,000,000 | PASS / FAIL |
| Mietsachschaden | {included per coverage_components} | Included | PASS / FAIL |
| Schlusselverlust | {included per coverage_components} | Included | PASS / FAIL |
| Gefalligkeitsschaden | {included per coverage_components} | Included | PASS / FAIL |
| Forderungsausfalldeckung | {included per coverage_components} | Included | PASS / FAIL |
| Selbstbeteiligung | €{from policy notes} | €0–150 | INFO |
```

Source: haftpflicht.md Coverage Benchmarks section

### Cancellation Alert Banner

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 CANCELLATION DEADLINE ALERT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
You have {X} days to cancel your {Type} policy.
Deadline: {deadline_date} ({kuendigungsfrist_months} months before renewal on {renewal_date})

To cancel: contact {provider} in writing before {deadline_date}.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Sonderkündigungsrecht Banner

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 SONDERKÜNDIGUNGSRECHT OPEN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
You have an extraordinary cancellation right currently open
for your {Type} policy. This window typically lasts 4 weeks
from the triggering event.

Review your last insurer correspondence to confirm the trigger
date and act before the window closes.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Research Agent Task Prompt (non-health generic)

```markdown
You are the finyx-insurance-research-agent. Research current market conditions for {insurance_type} insurance.

insurance_type: {slug}
current_year: {CURRENT_YEAR}
current_premium_monthly: {from profile if known, else omit}
[For kfz only:] sf_klasse: {collected value}
[For kfz only:] vehicle_age_years: {collected value}
[For kfz only:] annual_km: {collected value}

Return your output wrapped in <insurance_research_result> tags.
```

### Risikoleben Benchmark Computation

```markdown
From profile:
  gross_income = countries.germany.gross_income
  mortgage_balance = [check investor data for outstanding mortgage — if absent, use null]

recommended_sum = max(gross_income * 3, mortgage_balance or 0)
minimum_sum = gross_income * 3

Coverage adequacy check:
  PASS: coverage_amount >= recommended_sum
  MARGINAL: minimum_sum <= coverage_amount < recommended_sum
  FAIL: coverage_amount < minimum_sum
  UNKNOWN: coverage_amount is null (no policy recorded)
```

Source: risikoleben.md Coverage Benchmarks section

### Kfz SF-Klasse Question

```markdown
AskUserQuestion: "What is your SF-Klasse (Schadenfreiheitsklasse)?
This is your no-claims bonus class — it directly affects your premium.
0 = new driver / no claims history, 35 = 35+ claim-free years.
A single at-fault accident typically drops you 4-6 classes.
Find it on your current policy document (Versicherungsschein)."
```

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| Monolithic insurance command | Router + sub-skill dispatch (Phase 18) | Clean separation; this phase fills the 6 missing sub-skills |
| Generic placeholder sub-skills | Per-type specialized sub-skills (this phase) | TYPE-01 through TYPE-06 become functional |
| No cancellation tracking | OPT-03 inline in each sub-skill | Surfaces deadline info at point of type-specific consultation |

---

## Environment Availability

Step 2.6: SKIPPED — Phase 21 creates Markdown prompt files only. No external tools, services, runtimes, CLIs, or databases are introduced. The research agent uses WebSearch (a Claude Code built-in tool), which requires no installation.

---

## Open Questions

1. **Profile field `vertrag_beginn` vs `start_date`**
   - What we know: CONTEXT.md mentions `vertrag_beginn` as a cancellation tracking field; portfolio.md schema uses `start_date`
   - What's unclear: Whether `vertrag_beginn` was an early name that was renamed to `start_date` during Phase 19 implementation
   - Recommendation: Use `start_date` — it is the canonical field name in the portfolio.md schema (Phase 0 policy object construction). If `vertrag_beginn` also exists in some profiles, check both with fallback.

2. **Kfz user_city for Regionalklasse**
   - What we know: Research agent accepts `user_city` param for Kfz Regionalklasse localization; this is in kfz.md's rating factors
   - What's unclear: Where in the profile user city is stored
   - Recommendation: Check `identity.city` or `identity.residence_city` or any city-like field in profile. If absent, omit the param — agent handles missing params gracefully.

3. **Risikoleben mortgage balance location in profile**
   - What we know: risikoleben.md benchmark formula uses `outstanding_mortgage_balance`; profile has investor data
   - What's unclear: Exact field path for mortgage balance in `.finyx/profile.json`
   - Recommendation: Check `investor.mortgageBalance`, `investor.outstanding_loan`, or any field containing "mortgage" or "loan". If not found, use `gross_income × 3` only and note: "No mortgage data in profile — benchmark based on income multiple only."

---

## Sources

### Primary (HIGH confidence)

- `skills/insurance/sub-skills/health.md` — template pattern; all phase names, disclaimer placement, agent spawn format, error handling patterns
- `skills/insurance/sub-skills/portfolio.md` — second template; policy data check pattern, banner formats, write-only-in-phase-0 pattern
- `skills/insurance/agents/finyx-insurance-research-agent.md` — agent interface contract; Task prompt format, output XML tags, §34d GewO compliance behavior
- `skills/insurance/references/germany/haftpflicht.md` — benchmarks, cancellation rules, field extraction schema
- `skills/insurance/references/germany/hausrat.md` — benchmarks, cancellation rules, Unterversicherungsverzicht logic
- `skills/insurance/references/germany/kfz.md` — coverage tiers, SF-Klasse, rating factors, cancellation rules
- `skills/insurance/references/germany/rechtsschutz.md` — module structure, Wartezeit, service-based coverage_amount
- `skills/insurance/references/germany/zahnzusatz.md` — Staffelung, GKV Festzuschuss context, Wartezeit
- `skills/insurance/references/germany/risikoleben.md` — benchmark formula, health underwriting note, fixed-term renewal behavior
- `skills/insurance/SKILL.md` — router dispatch contract, allowed-tools list, sub-skill file contract

---

## Metadata

**Confidence breakdown:**
- Sub-skill file contract: HIGH — verified from two existing implementations (health.md, portfolio.md)
- Per-type benchmarks: HIGH — verified from 6 reference docs created in Phase 19
- Research agent interface: HIGH — verified from agent file
- Cancellation tracking logic: HIGH — all cancellation rules present in reference docs; profile schema verified from portfolio.md
- Router integration: HIGH — SKILL.md already wires all 6 slugs

**Research date:** 2026-04-12
**Valid until:** 2026-05-12 (stable codebase — conventions don't change between phases)
