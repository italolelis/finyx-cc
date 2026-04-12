---
name: finyx-insurance
description: Insurance advisor for Germany — routes to per-type sub-skills based on insurance type
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
  - WebSearch
  - WebFetch
disable-model-invocation: true
---

<objective>
Route the user's insurance question to the appropriate sub-skill advisor.
This skill:
1. Detects the insurance type from user input or asks when ambiguous
2. Loads and executes the matched sub-skill from ${CLAUDE_SKILL_DIR}/sub-skills/
3. Preserves the /finyx:insurance entry point for all insurance types
</objective>

<execution_context>
${CLAUDE_SKILL_DIR}/references/disclaimer.md
@.finyx/profile.json
</execution_context>

<process>

## Phase 0: Type Detection

Inspect `$ARGUMENTS` (the user's input after `/finyx:insurance`).

Keyword map (case-insensitive match against user input):
- "health", "kranken", "pkv", "gkv", "krankenversicherung" → sub-skill: health

If matched: set `sub_skill_type` to the matched value and proceed to Phase 1.

If not matched (no input or unrecognized type):
Use AskUserQuestion with singleSelect:
"Which insurance type would you like advice on?"
- Health insurance (Krankenversicherung — GKV vs PKV)

Map the answer:
- "Health insurance (Krankenversicherung — GKV vs PKV)" → sub_skill_type = "health"

## Phase 1: Sub-skill Dispatch

Read `${CLAUDE_SKILL_DIR}/sub-skills/${sub_skill_type}.md`.

Follow all instructions in the loaded file from its Phase 0 onward.

</process>

<error_handling>

**Unknown insurance type (sub-skill file not found):**
```
ERROR: No sub-skill found for type "[sub_skill_type]".
Currently available insurance types:
  - health (Krankenversicherung — GKV vs PKV)

Run /finyx:insurance health  (or use the type selection menu)
```

**No profile found:**
```
ERROR: No financial profile found.
Run /finyx:profile first to complete your financial profile, then retry.
```

</error_handling>

<notes>

## Router Scope

This SKILL.md is a pure dispatcher — it contains NO insurance-type-specific logic. All advisory content lives in sub-skill files under `${CLAUDE_SKILL_DIR}/sub-skills/`.

## Tool Permissions

The `allowed-tools` list is the union of all tools required by any sub-skill. Tool permissions are set at skill load time (YAML frontmatter), so the router must declare every tool any sub-skill may use. Current sub-skill tool requirements:
- health.md: Read, Bash, Write, Task, AskUserQuestion, WebSearch, WebFetch

When adding new sub-skills in future phases, verify their tool requirements and expand `allowed-tools` if needed.

## Sub-skill File Contract

Sub-skill files (e.g., `sub-skills/health.md`) must:
- Be plain Markdown — NO YAML frontmatter (sub-skills are not registered commands)
- NOT include an `<execution_context>` block (the router's execution_context already loads disclaimer.md and profile.json)
- Start with a `# [Type] Insurance Sub-skill` heading
- Contain their own `<objective>`, `<process>`, `<error_handling>`, and `<notes>` sections
- Use `${CLAUDE_SKILL_DIR}` for all reference and agent path references (resolves to `skills/insurance/`)

</notes>
