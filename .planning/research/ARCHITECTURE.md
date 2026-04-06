# Architecture Patterns

**Domain:** Multi-domain personal finance advisor CLI (Claude Code slash-command plugin)
**Researched:** 2026-04-06
**Confidence:** HIGH (based on official Claude Code docs + existing codebase analysis)

---

## Recommended Architecture

Extend the existing meta-prompting system. Do not introduce an application framework. The proven pattern — Markdown prompts + sub-agent `.md` files + JSON profile + reference docs — scales directly to new financial domains without structural change.

The core addition is a **shared user financial profile** (`.finyx/profile.json`) that all specialist agents read, and a **country-scoped reference doc tree** that makes multi-country rules a file layout concern, not a logic concern.

---

## Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| `commands/fin/` | User-facing slash-commands (`/fin:*`). Orchestrate workflows, spawn agents, read/write state | Agents (via `Agent` tool), `finyx/references/`, `.finyx/profile.json` |
| `agents/` | Specialist sub-agents (tax-de, tax-br, investment, portfolio, pension, real-estate) | Read `.finyx/profile.json` + country references injected by orchestrating command; return structured output |
| `finyx/references/<country>/` | Country-specific tax rules, investment rules, pension rules as versioned Markdown knowledge docs | Injected into command context via `@path`; passed to agents in task prompt |
| `finyx/references/global/` | Country-agnostic rules: ETF concepts, broker comparison criteria, risk frameworks | Same injection pattern |
| `finyx/skills/` | Reusable domain knowledge blocks (e.g., `de-tax-math.md`, `br-ir-rules.md`) loaded into agents via `skills:` frontmatter field | Agents that declare them in frontmatter |
| `.finyx/profile.json` | Runtime user financial profile — single source of truth for all agents | Written by `/fin:profile`; read by every subsequent command and agent |
| `.finyx/STATE.md` | Progress/session tracker, updated by each command | All commands |
| `.finyx/portfolio/` | User's holdings, transactions, allocation snapshots written by portfolio commands | Portfolio agent, rebalance command |
| `bin/install.js` (extended) | Copies `commands/fin/`, `agents/`, `finyx/` into `~/.claude/` alongside existing `immo/` | End users via `npx finyx` |

**Boundary rule:** Agents never write to profile or state directly. Only orchestrating commands do. Agents receive their inputs in the Task prompt and return structured Markdown output. This keeps agents stateless and re-invocable.

---

## Data Flow

### Profile Bootstrap (one-time)

```
/fin:profile
  → AskUserQuestion: country, income, tax class, family status, risk tolerance, goals
  → AskUserQuestion: current holdings, broker, pension status
  → Write .finyx/profile.json
  → Write .finyx/STATE.md (profile: complete)
```

### Domain Command (e.g., /fin:tax)

```
/fin:tax
  → Phase 1: Read .finyx/profile.json (detect country)
  → Phase 2: Load country references via @path
      Germany: @~/.claude/finyx/references/germany/income-tax.md
               @~/.claude/finyx/references/germany/investment-tax.md
      Brazil:  @~/.claude/finyx/references/brazil/ir-investments.md
               @~/.claude/finyx/references/brazil/darf-rules.md
  → Phase 3: Spawn specialist agent via Agent tool
      Task prompt includes: profile JSON + reference doc content
      Agent: finyx-tax-de-agent OR finyx-tax-br-agent (selected by command based on profile.country)
  → Phase 4: Receive agent output, format, write .finyx/output/TAX-ADVICE-[DATE].md
```

### Market Data Command (e.g., /fin:market)

```
/fin:market [ticker]
  → Phase 1: WebSearch "[ticker] price site:finance.yahoo.com OR site:tradingview.com"
             WebFetch structured data from result
  → Phase 2: WebSearch "[ticker] news last 7 days"
  → Phase 3: Spawn finyx-investment-agent with market data + profile in prompt
  → Phase 4: Return analysis
```

**Market data decision:** No runtime npm dependency. Use WebSearch + WebFetch as primary. This preserves zero-dependency architecture. If a user has an API key (Alpha Vantage, FMP), it goes in `.finyx/profile.json` under `apiKeys` and commands check for it as an enhancement path. Web search is the guaranteed fallback.

### Cross-Agent Shared Context Pattern

Sub-agents do not share a live context. Shared state is via files. The pattern:

1. Command reads `.finyx/profile.json` at start of every invocation
2. Command selects which reference docs to inject (based on `profile.country`)
3. Command builds a structured task prompt containing: profile summary + relevant reference sections + specific question
4. Agent runs in isolated context window with that task prompt as its full context
5. Agent returns Markdown output; command writes it to `.finyx/output/`

This is the correct pattern for this architecture. Sub-agents cannot access files autonomously unless the command grants them the task prompt — inject what each agent needs explicitly.

**Persistent memory for agents** (optional, Phase 2+): Claude Code's `memory: project` field on agent frontmatter gives each specialist agent a `MEMORY.md` at `.claude/agent-memory/<agent-name>/`. Use this for agents that accumulate user-specific insights across sessions (e.g., the tax agent remembering the user's deduction history). Configure `memory: local` so it doesn't commit to version control.

---

## Multi-Country Reference Doc Organization

Organize strictly by country code. Each domain under each country mirrors the same file names — commands select the right path based on `profile.country`.

```
finyx/references/
  germany/
    income-tax.md          # Einkommensteuer brackets, Soli, Kirchensteuer
    investment-tax.md      # Sparerpauschbetrag, Vorabpauschale, Teilfreistellung
    real-estate-tax.md     # AfA, Grunderwerbsteuer (extracted from immo)
    pension.md             # Riester, Rürup, bAV rules
    brokers.md             # TR, Scalable, ING, comdirect comparison matrix
  brazil/
    income-tax.md          # IR brackets, deductions
    investment-tax.md      # IR on stocks/ETFs/FIIs, come-cotas, isenção R$20k
    real-estate-tax.md     # ITBI, IRPF on ganho de capital
    pension.md             # INSS, previdência privada, PGBL vs VGBL
    brokers.md             # XP, NuInvest, BTG comparison matrix
    darf.md                # DARF calculation rules, due dates
  global/
    etf-fundamentals.md    # How ETFs work, TER, tracking error
    risk-frameworks.md     # Risk tolerance definitions used in profile
    portfolio-theory.md    # Diversification, rebalancing concepts
```

**Country selection in commands:** Commands use a single conditional block:

```
IF profile.country == "DE" → inject finyx/references/germany/* relevant files
IF profile.country == "BR" → inject finyx/references/brazil/* relevant files
IF profile.country == "BOTH" → inject both (expat case)
```

This is a file-path switch, not a rule engine. The "rule engine" is the LLM reading the injected docs. No code needed.

**Versioning:** Each reference file carries a `Tax Year: YYYY` header. When tax rules change, update the file and bump the year. Commands can surface this to the user ("Using 2025 German investment tax rules").

---

## Extending Slash-Command Architecture for New Domains

Follow the existing pattern exactly. Each new domain requires:

1. **Command file:** `commands/fin/[domain].md` — orchestrates the workflow, handles profile loading, selects country references, spawns agent(s)
2. **Agent file(s):** `agents/finyx-[domain]-[country]-agent.md` — specialist prompt, tools restricted to what it needs, `skills:` list for preloaded domain knowledge
3. **Reference docs:** `finyx/references/[country]/[domain].md` — knowledge that changes (tax rates, rules); put stable concepts in `finyx/skills/`
4. **Profile extension:** If the new domain needs new profile fields (e.g., pension contributions for pension domain), extend `.finyx/profile.json` schema and add questions to `/fin:profile`

No installer changes needed for new domains — `bin/install.js` already copies the entire `finyx/` tree.

---

## Component Interaction Map

```
User types /fin:tax
      |
      v
commands/fin/tax.md (orchestrator)
  reads: .finyx/profile.json
  injects: @finyx/references/germany/income-tax.md
           @finyx/references/germany/investment-tax.md
  spawns: Agent(finyx-tax-de-agent)
             |
             v
         agents/finyx-tax-de-agent.md
           skills: [de-tax-math, sparerpauschbetrag-rules]
           tools: [Read, WebSearch]
           memory: local
           returns: structured tax analysis Markdown
      |
      v
commands/fin/tax.md writes: .finyx/output/TAX-ADVICE-[DATE].md
                    prints: formatted advice to user
```

---

## Build Order Implications

1. **Profile first.** `.finyx/profile.json` is the dependency for everything else. `/fin:profile` command must exist before any specialist command can function. Build this in Phase 1.

2. **Country references before agents.** Agents are only useful when their reference docs exist. Write Germany tax reference docs before building the German tax agent. Same for Brazil.

3. **One country before two.** Build Germany end-to-end (profile → references → agent → command) then add Brazil as a parallel path. Do not build both simultaneously — country-specific bugs are easier to isolate one at a time.

4. **Real estate reuse before expansion.** The existing `immo/references/germany/tax-rules.md` can be imported into the new finyx structure. Extract and reorganize rather than rewrite. This is Phase 1 work.

5. **Market data last.** WebSearch-based market data adds no new structural patterns — it's just a command with WebSearch/WebFetch tools. Build after the specialist agents are stable, since it depends on the investment agent for analysis.

6. **Agent persistent memory is additive.** Add `memory: local` to agents after their core behavior is validated. Do not start with memory enabled — it complicates debugging during development.

---

## Scalability Considerations

| Concern | Current Scale (1 user) | Multi-contributor Scale | Country Expansion |
|---------|----------------------|------------------------|-------------------|
| Reference doc maintenance | One person updates manually | Each contributor owns a country dir; PRs per tax year | Add `countries/XX/` dir, no core changes |
| Agent proliferation | 6-8 agents, manageable | Name convention `finyx-[domain]-[country]` prevents collision | New country = new set of country-specific agents |
| Profile schema evolution | Add fields freely | Schema version field in profile.json | Backward compat: commands check field existence before using |
| Context window size | 5-6 reference docs per invocation, well within limits | Prune `@path` includes per command to only what's needed | Per-country scope keeps injection focused |

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: God Profile Command
**What:** A single `/fin:profile` command that tries to gather every field for every domain and every country upfront.
**Why bad:** 20+ questions at init time creates friction; most fields won't be needed in early sessions.
**Instead:** Gather core fields at init (country, income, tax class, risk). Add domain-specific fields lazily when a user first invokes that domain's command — "To give you pension advice, I need a few more details."

### Anti-Pattern 2: Agent-to-Agent Communication
**What:** Having the tax agent call the investment agent directly.
**Why bad:** Sub-agents cannot spawn other sub-agents in Claude Code. Attempting this silently fails.
**Instead:** The orchestrating command is the hub. If a workflow needs output from two agents, the command runs them sequentially, collects both outputs, and synthesizes.

### Anti-Pattern 3: Embedding Country Logic in Command Code
**What:** `if country == DE: use_rate(0.25) else: use_rate(0.15)` in command prompts.
**Why bad:** Tax rates change yearly; hardcoded values require command edits, not just reference doc updates.
**Instead:** All rates live in reference docs. Command injects the doc. Agent reads the doc. Zero rate literals in command files.

### Anti-Pattern 4: One Fat Reference File Per Country
**What:** `germany.md` containing all German rules — tax, pension, brokers, real estate — in one file.
**Why bad:** Entire file gets injected even when only pension rules are needed, bloating context.
**Instead:** One file per domain per country. Commands inject only the files relevant to the current task.

### Anti-Pattern 5: Runtime npm Dependencies for Market Data
**What:** Adding `yahoo-finance2` or `node-fetch` as production dependencies.
**Why bad:** Breaks zero-dependency architecture; creates install-time failures and maintenance burden.
**Instead:** WebSearch + WebFetch for all market data. Store optional API keys in profile.json and use Bash `curl` calls if keys exist — this keeps Node.js out of the data path entirely.

---

## Sources

- Claude Code Sub-Agents official docs (HIGH confidence): https://code.claude.com/docs/en/sub-agents
- Existing IMMO codebase analysis (`agents/`, `commands/immo/`, `immo/references/`) — direct inspection (HIGH confidence)
- `.planning/PROJECT.md` and `.planning/codebase/ARCHITECTURE.md` — project context (HIGH confidence)
- Multi-agent orchestration patterns: https://shipyard.build/blog/claude-code-multi-agent/ (MEDIUM confidence)
- yahoo-finance2 npm: https://www.npmjs.com/package/yahoo-finance2 (MEDIUM confidence — unofficial API, stability risk)
