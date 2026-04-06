# Testing Patterns

**Analysis Date:** 2026-04-06

## Test Framework

**Runner:** None detected

**Assertion Library:** None detected

**Test Files Found:** None

```bash
# No test runner configured
# package.json has no test script, jest, vitest, or mocha dependency
```

---

## Current State

This codebase has **no automated tests**. There is no `jest.config.*`, `vitest.config.*`, `.mocharc*`, or any `*.test.*`/`*.spec.*` files. The `package.json` defines no test dependencies and no `scripts.test` entry.

---

## Testable Surface Area

The codebase has two testable components:

### 1. JavaScript Installer (`bin/install.js`)

The installer is a single CommonJS module with these pure-ish functions:

| Function | Inputs | Side Effects |
|----------|--------|-------------|
| `getClaudeDir()` | `process.env.CLAUDE_CONFIG_DIR`, `os.homedir()` | None |
| `copyWithPathReplacement(srcDir, destDir, pathPrefix)` | File paths, string prefix | Reads/writes filesystem |
| `install(isGlobal)` | Boolean | Filesystem writes |
| `uninstall(isGlobal)` | Boolean | Filesystem deletes |

`getClaudeDir()` is the only fully pure function. The rest require filesystem mocking.

### 2. Claude Command Definitions (`commands/immo/*.md`, `agents/*.md`)

These are Markdown prompts executed by Claude at runtime. They are not testable with standard unit test frameworks. Correctness is validated by:
- Manual invocation in Claude Code
- Reviewing generated output files (`.immo/analysis/`, `.immo/research/`, `.immo/output/`)

---

## If Tests Were Added

Based on the codebase structure, the recommended approach would be:

**Framework:** Vitest (ESM-compatible, fast, minimal config) or Jest with CommonJS

**What to test in `bin/install.js`:**

```js
// getClaudeDir() - pure function, env-dependent
describe('getClaudeDir', () => {
  it('returns CLAUDE_CONFIG_DIR when set', () => {
    process.env.CLAUDE_CONFIG_DIR = '/custom/path';
    expect(getClaudeDir()).toBe('/custom/path');
  });

  it('falls back to ~/.claude when env not set', () => {
    delete process.env.CLAUDE_CONFIG_DIR;
    expect(getClaudeDir()).toBe(path.join(os.homedir(), '.claude'));
  });
});

// copyWithPathReplacement - requires fs mocking
describe('copyWithPathReplacement', () => {
  it('replaces ~/.claude/ with pathPrefix in .md files', () => {
    // mock fs.readdirSync, fs.readFileSync, fs.writeFileSync
  });

  it('copies non-.md files without modification', () => {
    // assert fs.copyFileSync called, not writeFileSync with replacement
  });
});
```

**Test file location:** Co-located with source, e.g., `bin/install.test.js`

**Config file:** `vitest.config.js` or `jest.config.js` at root

---

## Integration / Manual Testing

Since commands are prompt files, integration testing is manual:

1. Install locally: `node bin/install.js --local`
2. Verify copied files exist in `.claude/commands/immo/`
3. Verify path replacement: `grep -r "~/.claude" .claude/` should return nothing
4. Verify agents copied: `ls .claude/agents/immo-*.md`
5. Verify VERSION file written: `cat .claude/immo/VERSION`
6. Uninstall: `node bin/install.js --local --uninstall`
7. Verify cleanup: confirm `.claude/commands/immo/` removed

---

## Coverage

**Current:** 0% — no test runner exists

**Recommended minimum for `bin/install.js`:**
- `getClaudeDir()` — 100% (2 branches: env set, env not set)
- `copyWithPathReplacement()` — directory recursion, .md replacement, non-.md copy
- CLI argument parsing — `--global`, `--local`, `--uninstall`, `--help`, conflict detection

---

## Quality Gates

No linting, no formatting, no pre-commit hooks configured. No `.eslintrc*`, `.prettierrc*`, or `biome.json` present.

The only quality enforcement is npm's `files` field in `package.json`, which restricts published artifacts to `bin/`, `commands/`, `immo/`, `agents/`, and `scripts/`.

---

*Testing analysis: 2026-04-06*
