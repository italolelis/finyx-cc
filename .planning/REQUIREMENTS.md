# Requirements: Finyx v2.0

**Defined:** 2026-04-12
**Core Value:** A single AI-powered financial advisor that knows your full financial picture and gives integrated, country-aware advice

## v2.0 Requirements

Requirements for the Plugin Architecture milestone. Each maps to roadmap phases.

### Plugin Foundation

- [ ] **PLUG-01**: `.claude-plugin/plugin.json` manifest exists with correct name and metadata for third-party plugin installation
- [ ] **PLUG-02**: Directory restructured from `commands/finyx/*.md` to `skills/<name>/SKILL.md` format with naming that preserves `/finyx:*` syntax
- [ ] **PLUG-03**: All `@~/.claude/finyx/references/` path includes replaced with `${CLAUDE_SKILL_DIR}/references/` across all skills

### Skill Conversion

- [ ] **SKILL-01**: All 17 commands converted to SKILL.md files with proper frontmatter (name, description, allowed-tools)
- [ ] **SKILL-02**: Each advisory skill has `disable-model-invocation: true` to prevent finance vocab auto-trigger misfires
- [ ] **SKILL-03**: Every agent assigned to its owning skill under `skills/<name>/agents/` — no shared root-level agents
- [ ] **SKILL-04**: Each skill bundles its own reference docs under `skills/<name>/references/`
- [ ] **SKILL-05**: Skill trigger descriptions written for model detection (front-loaded, specific, 250 char max)

### Integration

- [ ] **INTG-01**: `.finyx/profile.json` access works from skill context with `~/.finyx/` global fallback
- [ ] **INTG-02**: Cross-skill integration for `finyx-insights` works when other finyx skills are co-installed
- [ ] **INTG-03**: Plugin installable as third-party via GitHub URL (standard `claude plugin install` flow)

### Cleanup

- [ ] **CLEAN-01**: Legacy `commands/finyx/` directory removed (skills replace all commands)
- [ ] **CLEAN-02**: Legacy `agents/` root directory removed (agents redistributed into skills)
- [ ] **CLEAN-03**: `bin/install.js` updated or removed to reflect new distribution model

## v2.1 Requirements

Deferred to future release.

- **DIST-01**: Submit to Anthropic official plugin directory
- **HOOK-01**: SessionStart hook for tax year staleness detection
- **INDV-01**: Individual skill installation support

## Out of Scope

| Feature | Reason |
|---------|--------|
| npm-only fallback distribution | Plugin system replaces npm as primary — clean break |
| claudes-kitchen submission | User's workplace plugin registry, not for Finyx distribution |
| Individual skill packages | Plugin is the install unit — users get all skills at once |
| Automated trading/execution | Advisory only — unchanged from v1.x |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| PLUG-01 | — | Pending |
| PLUG-02 | — | Pending |
| PLUG-03 | — | Pending |
| SKILL-01 | — | Pending |
| SKILL-02 | — | Pending |
| SKILL-03 | — | Pending |
| SKILL-04 | — | Pending |
| SKILL-05 | — | Pending |
| INTG-01 | — | Pending |
| INTG-02 | — | Pending |
| INTG-03 | — | Pending |
| CLEAN-01 | — | Pending |
| CLEAN-02 | — | Pending |
| CLEAN-03 | — | Pending |

**Coverage:**
- v2.0 requirements: 14 total
- Mapped to phases: 0
- Unmapped: 14 ⚠️

---
*Requirements defined: 2026-04-12*
*Last updated: 2026-04-12 after initial definition*
