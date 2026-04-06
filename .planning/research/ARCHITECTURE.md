# Architecture Research

**Domain:** Financial insights command — `/fin:insights` integration into Finyx slash-command architecture
**Researched:** 2026-04-06
**Confidence:** HIGH (based on direct codebase inspection)

---

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User: /fin:insights                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│              commands/finyx/insights.md                      │
│  Orchestrator: validates profile, loads refs, spawns agents  │
│                                                              │
│  <execution_context>                                         │
│    @.finyx/profile.json                                      │
│    @~/.claude/finyx/references/insights/benchmarks.md        │
│    @~/.claude/finyx/references/insights/scoring-rules.md     │
│    @~/.claude/finyx/references/disclaimer.md                 │
│  </execution_context>                                        │
└──────────────────────────┬──────────────────────────────────┘
                           │  Task tool (parallel)
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼───────┐ ┌────────▼──────┐ ┌────────▼──────────┐
│ insights-     │ │ insights-     │ │ insights-          │
│ allocation-   │ │ tax-scoring-  │ │ projection-        │
│ agent.md      │ │ agent.md      │ │ agent.md           │
│               │ │               │ │                    │
│ Income split  │ │ Unused allow- │ │ Net worth snap-    │
│ needs/wants/  │ │ ances, gaps   │ │ shot, goal pace,   │
│ savings bench │ │ across DE+BR  │ │ forward model      │
└───────┬───────┘ └────────┬──────┘ └────────┬──────────┘
        │                  │                  │
        └──────────────────▼──────────────────┘
                           │
        ┌──────────────────▼──────────────────┐
        │  insights.md synthesis phase         │
        │  - Merge FLAGS from all agents       │
        │  - Sort RECOMMENDATIONS by EUR impact│
        │  - Surface cross-domain links        │
        │  - Emit unified report               │
        └─────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Implementation |
|-----------|----------------|----------------|
| `commands/finyx/insights.md` | Orchestrator — validate profile, spawn 3 agents in parallel, synthesize unified report | Slash-command `.md`, `Task` in `allowed-tools` |
| `agents/insights-allocation-agent.md` | Classify income into needs/wants/savings/investments/debt; benchmark against country-aware norms | Agent `.md`, reads profile + benchmarks ref via injected context |
| `agents/insights-tax-scoring-agent.md` | Score tax efficiency 0-100; identify unused Sparerpauschbetrag, missed Riester/Rürup windows, BR DARF gaps | Agent `.md`, reads profile + existing DE/BR tax refs via injected context |
| `agents/insights-projection-agent.md` | Net worth snapshot; compound savings/investment rate to goal targets; calculate months-to-target | Agent `.md`, reads profile + scoring-rules ref via injected context |
| `finyx/references/insights/benchmarks.md` | Country-aware allocation benchmarks (50/30/20 adapted for DE/BR tax burden); net worth targets by age bracket | New reference doc, versioned with `tax_year` header |
| `finyx/references/insights/scoring-rules.md` | Tax efficiency rubric (0-100); allocation health scoring; goal pace thresholds; recommendation ranking formulas | New reference doc, versioned |

---

## Recommended Project Structure

New files only — no modifications to existing commands or agents.

```
commands/finyx/
└── insights.md                          # new orchestrator command

agents/
├── insights-allocation-agent.md         # new
├── insights-tax-scoring-agent.md        # new
└── insights-projection-agent.md         # new

finyx/references/
└── insights/
    ├── benchmarks.md                    # new — allocation benchmarks DE+BR
    └── scoring-rules.md                 # new — scoring thresholds and ranking logic
```

Existing files untouched: `tax.md`, `invest.md`, `pension.md`, `broker.md`, all existing agents, all existing reference docs.

---

## Architectural Patterns

### Pattern 1: Profile-Only Data Sourcing

**What:** `insights` reads `.finyx/profile.json` directly. It does not invoke or parse output from `tax`, `invest`, `pension`, or `broker`.

**When to use:** Always for `insights`. The advisor commands are interactive — they ask questions and fetch live data. `insights` must be non-interactive to produce a complete snapshot in one shot. The profile already contains the structured inputs all advisors use.

**Trade-offs:** Profile data may lag a fresh `/fin:invest` run. This is acceptable — `insights` is a health dashboard, not a live feed. If holdings are missing, emit a callout rather than failing.

**Key profile fields consumed by agents:**
```json
identity.cross_border
identity.family_status
countries.germany.gross_income
countries.germany.marginal_rate
countries.germany.tax_class
countries.germany.brokers[].holdings[]
countries.brazil.gross_income
countries.brazil.ir_regime
investor.liquidAssets
investor.monthlyCommitments
goals.risk_tolerance
goals.investment_horizon
goals.primary_goals
```

Holdings (`brokers[].holdings[]`) and pension contribution fields must be present for full insights. When absent, agents emit `FLAGS: INCOMPLETE — run /fin:invest to add holdings` and proceed with partial analysis.

### Pattern 2: Parallel Agent Decomposition

**What:** Three analytic concerns — allocation, tax efficiency, projection — are independently solvable from the same profile data. Spawn them in parallel via `Task`, collect structured outputs, synthesize in the orchestrator.

**When to use:** When agents have no inter-agent dependency and can work from the same inputs. All three insights agents qualify.

**Trade-offs:** Adds 3 agent files to maintain. The alternative (monolithic `insights.md`) degrades analysis quality as the prompt becomes too long. This split mirrors how `analyze.md` delegates to `finyx-analyzer-agent.md`.

**Synthesis contract — each agent returns this structure:**
```
SECTION: allocation|tax|projection
SCORE: 0-100 or N/A
FLAGS:
- [issue description] | IMPACT: €X/year | PRIORITY: high|medium|low
RECOMMENDATIONS:
1. [Action] | IMPACT: €X/year | EFFORT: low|medium|high
```
The orchestrator merges all FLAGS, sorts RECOMMENDATIONS descending by EUR impact, and surfaces cross-domain links where one recommendation affects multiple sections.

### Pattern 3: Reference Doc Injection via execution_context

**What:** Benchmarks and scoring rules live in versioned reference docs loaded via `@` includes in `<execution_context>`. Agents receive them as injected context from the orchestrator — they do not use `@` includes themselves.

**When to use:** Any knowledge that is country-specific, time-varying, or needs versioning. Tax benchmarks (Sparerpauschbetrag amount, pension contribution limits) change yearly.

**Trade-offs:** Docs must be kept current. Use same `tax_year: YYYY` metadata pattern as `germany/tax-investment.md` so commands can surface staleness warnings.

---

## Data Flow

### Request Flow

```
User: /fin:insights
    |
    v
Phase 1: Validate .finyx/profile.json exists
    bash: [ -f .finyx/profile.json ] || abort with "Run /fin:profile first"
    |
    v
Phase 2: Detect active countries
    Germany active if: countries.germany.tax_class != null
    Brazil active if:  countries.brazil.ir_regime != null
    Cross-border if:   identity.cross_border == true
    |
    v
Phase 3: Spawn agents in parallel via Task
    Task(insights-allocation-agent)  ─┐
    Task(insights-tax-scoring-agent) ─┼─ parallel, same profile context
    Task(insights-projection-agent)  ─┘
    |
    v
Phase 4: Collect agent outputs
    Parse SECTION / SCORE / FLAGS / RECOMMENDATIONS from each
    |
    v
Phase 5: Cross-advisor synthesis
    - Merge all FLAGS into unified list
    - Sort all RECOMMENDATIONS by EUR impact descending
    - Identify cross-domain links:
        e.g., "Increasing Rürup saves €X in tax AND closes pension gap"
        e.g., "Equity overweight increases risk beyond risk_tolerance"
    - Compute composite Financial Health Score (weighted average of agent scores)
    |
    v
Phase 6: Emit unified report
    ┌─────────────────────────────────────┐
    │  FINYX ► FINANCIAL INSIGHTS         │
    ├─────────────────────────────────────┤
    │  Financial Health Score: XX/100     │
    │  Income Allocation breakdown        │
    │  Tax Efficiency: XX/100 + gaps      │
    │  Net Worth: €X → Goal in Y months   │
    │  ─────────────────────────────────  │
    │  Top Recommendations (ranked)       │
    │  1. [action] | €X/year impact       │
    │  2. ...                             │
    ├─────────────────────────────────────┤
    │  [legal disclaimer]                 │
    └─────────────────────────────────────┘
```

### Key Data Flows

1. **Profile → Agents.** Profile is passed as context when spawning each Task. Agents do not issue Bash reads of `.finyx/profile.json` autonomously — they receive the full profile via the orchestrator's injected context. This is how `finyx-analyzer-agent` and `finyx-reporter-agent` already work.

2. **Recommendations ranking.** Each agent emits recommendations with estimated EUR impact. Orchestrator sorts the merged list descending by impact. Cross-domain links are identified by the orchestrator in Phase 5 — not by individual agents — because agents lack visibility into each other's output.

3. **Missing data path.** When a profile section is absent (no holdings, no pension data), the relevant agent emits `FLAGS: INCOMPLETE`. Orchestrator converts this to a callout block in the report: "Add your holdings by running `/fin:invest` for full portfolio analysis." Analysis proceeds with available data; the report clearly marks which sections are partial.

4. **Country scoping.** Tax-scoring agent runs DE logic only if Germany active, BR logic only if Brazil active, both if `cross_border: true`. Same pattern used by `tax.md`, `invest.md`, `pension.md`.

---

## Integration Points

### Existing Commands — No Modification Required

| Command | Data Available in Profile | What insights Consumes |
|---------|--------------------------|------------------------|
| `finyx:tax` | `countries.*.marginal_rate`, `countries.*.tax_class` | Tax efficiency scoring inputs |
| `finyx:invest` | `countries.*.brokers[].holdings[]`, `goals.risk_tolerance` | Allocation analysis, rebalancing gap detection |
| `finyx:pension` | Pension contributions (if written back to profile on consent) | Pension gap in projection |
| `finyx:broker` | `countries.*.brokers[]` | Tax-reporting quality contribution to score |
| `finyx:profile` | Writes `.finyx/profile.json` | Primary data source for all three agents |

No existing command needs modification. `insights` is purely additive.

### New Reference Docs Needed

| File | Content | Versioning |
|------|---------|------------|
| `finyx/references/insights/benchmarks.md` | Income split benchmarks by country + family status; net worth targets by age/income bracket; savings rate norms; ETF allocation targets | `tax_year: 2025` header |
| `finyx/references/insights/scoring-rules.md` | Tax efficiency score rubric; allocation health scoring; goal pace thresholds (on-track/at-risk/off-track); cross-domain link detection rules; recommendation impact estimation guidance | `version: 1.0` header |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| `insights.md` ↔ agents | Task tool with injected profile + refs | Same pattern as `analyze.md` ↔ `finyx-analyzer-agent` |
| Agents ↔ reference docs | Injected via orchestrator's `<execution_context>` | Agents do not issue `@` includes directly |
| `insights.md` ↔ `.finyx/profile.json` | Bash existence check + Claude context load | Read-only — `insights` never writes to profile |
| `insights-tax-scoring-agent` ↔ existing tax refs | Reuse `germany/tax-investment.md` and `brazil/tax-investment.md` via context injection | No duplication needed; these docs already have the allowance/rate data |

---

## Build Order

Dependencies drive this order. Each step unblocks the next.

1. **`finyx/references/insights/benchmarks.md`**
   - No dependencies. Define DE+BR income allocation benchmarks and net worth targets.
   - Unblocks allocation agent and sets reference baseline for scoring agent.

2. **`finyx/references/insights/scoring-rules.md`**
   - Depends on benchmarks existing (scoring references benchmark thresholds).
   - Defines rubrics used by all three agents.

3. **`agents/insights-allocation-agent.md`**
   - Depends on benchmarks ref. Independently testable: feed profile + benchmarks, verify SECTION/SCORE/FLAGS/RECOMMENDATIONS output format.

4. **`agents/insights-tax-scoring-agent.md`**
   - Depends on scoring-rules ref. Reuses existing `germany/tax-investment.md` and `brazil/tax-investment.md` — no new tax refs needed.
   - Independently testable with a profile that has `marginal_rate` and `tax_class`.

5. **`agents/insights-projection-agent.md`**
   - Depends on scoring-rules ref for on-track/at-risk thresholds. Can run with minimal profile (income + liquidAssets + goals).

6. **`commands/finyx/insights.md`**
   - Depends on all three agents and both reference docs existing.
   - Wire agents, define synthesis logic, add to `bin/install.js` copy paths.
   - This is the only file that touches `bin/install.js` (to ensure `finyx/references/insights/` gets copied on install).

---

## Anti-Patterns

### Anti-Pattern 1: Calling Other Commands from insights

**What people do:** Invoke `/fin:tax`, `/fin:invest` from inside `insights` to get "fresh" data.

**Why it's wrong:** Those commands are interactive — they ask questions and fetch live API data. Chaining them makes `insights` non-deterministic, slow, and unrunnable without a live session. The profile already contains the structured inputs they use.

**Do this instead:** Read `.finyx/profile.json` directly. When data is stale or missing, emit a prompt to run the relevant advisor before re-running insights.

### Anti-Pattern 2: Monolithic insights.md Without Agents

**What people do:** Put all three analytic concerns (allocation, tax, projection) in a single prompt with no agent decomposition.

**Why it's wrong:** A single prompt covering DE+BR tax rules, allocation benchmarks, scoring rubrics, and projection math will exceed the quality threshold where Claude reasons well. Three distinct analytic frames need focused contexts.

**Do this instead:** Three agents, each with one responsibility. The orchestrator synthesizes — it does not compute.

### Anti-Pattern 3: Hardcoding Benchmarks in Agent Prompts

**What people do:** Embed `50% needs / 30% wants / 20% savings` directly in agent prompt text.

**Why it's wrong:** Benchmarks are country-specific (DE/BR tax burden shifts the ratios), age-adjusted, and change over time. Hardcoded values are invisible and unversioned.

**Do this instead:** `finyx/references/insights/benchmarks.md` with `tax_year` metadata, loaded via `<execution_context>`. Same pattern as `germany/tax-investment.md`.

### Anti-Pattern 4: Writing insights Output to Profile

**What people do:** Have `insights` update `profile.json` with scores or recommendations.

**Why it's wrong:** Scores are computed artifacts, not source data. Writing them into the profile creates stale computed state that could corrupt future analyses.

**Do this instead:** Output is conversational only — no files written. If users want to save a report, add an optional `Write .finyx/output/INSIGHTS-[DATE].md` with explicit user consent, same pattern as other advisor commands.

---

## Sources

- Direct inspection: `commands/finyx/tax.md`, `invest.md`, `pension.md`, `broker.md`, `profile.md`
- Direct inspection: `agents/finyx-analyzer-agent.md`, `finyx-reporter-agent.md`
- Direct inspection: `.finyx/profile.json` schema (full field inventory)
- Direct inspection: `finyx/references/` structure (germany/, brazil/, disclaimer.md)
- Pattern reference: orchestrator→agent delegation in `analyze.md` → `finyx-analyzer-agent`

---
*Architecture research for: Finyx /fin:insights command — v1.1 milestone*
*Researched: 2026-04-06*
