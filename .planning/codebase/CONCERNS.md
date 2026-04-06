# Codebase Concerns

**Analysis Date:** 2026-04-06

---

## Tech Debt

**Germany-Only Tax Engine:**
- Issue: All tax calculation logic (Sonder-AfA, Solidaritätszuschlag, Nebenkosten by state, Spekulationsfrist) is hard-wired for Germany only. `country: "Portugal" | "Spain" | "Other"` paths in `commands/immo/init.md` explicitly state "basic analysis only" and "country-specific tax rules are not yet implemented."
- Files: `commands/immo/init.md`, `immo/references/germany/tax-rules.md`, `immo/references/methodology.md`, `agents/immo-analyzer-agent.md`
- Impact: Any user investing in non-Germany EU markets gets degraded, unvalidated output without warning in the analysis pipeline. Commands `analyze`, `filter`, `stress-test`, `report` all assume German tax mechanics silently.
- Fix approach: Add a country guard at the start of each analysis command; surface a clear "Germany only" notice; stub non-Germany tax modules before expanding.

**Tax Rule Date-Lock:**
- Issue: `immo/references/germany/tax-rules.md` specifies "Income Tax Brackets (2024+)" and hardcodes the Sonder-AfA §7b eligibility cutoff as "building permit application before January 1, 2027." No version or validity period is stored in the file itself or the config schema.
- Files: `immo/references/germany/tax-rules.md`, `immo/templates/config.json`
- Impact: Tax thresholds and AfA eligibility rules will silently go stale as German law changes. Users running IMMO after 2027 may receive incorrect tax benefit calculations with no indication of staleness.
- Fix approach: Add a `validUntil` or `lastVerified` field to `tax-rules.md` frontmatter; add a staleness check in `analyze.md` Phase 1 validation.

**Vacancy Assumption Hardcoded to Zero:**
- Issue: `immo/templates/config.json` sets `"vacancy": 0` and `immo/references/methodology.md` Rule 5 explicitly labels it "Already conservative." The stress-test command models 3 months every 2 years (12.5% loss) as a scenario only — the base analysis uses 0%.
- Files: `immo/templates/config.json`, `commands/immo/stress-test.md`, `immo/references/methodology.md`
- Impact: Base analysis systematically overstates expected cashflow. A 0% vacancy assumption over 10 years is not conservative; it is optimistic for most rental markets.
- Fix approach: Default `vacancy` to 2-4% in `config.json` template; document the reasoning; treat 0% as a best-case scenario in `stress-test.md`.

**Declining Interest Not Modeled in Years 5-10:**
- Issue: In `commands/immo/analyze.md` Phase 4.4, Years 5-10 interest is approximated as "85% of Y1" — a flat estimate rather than a proper amortization schedule. The same pattern appears in `agents/immo-analyzer-agent.md` Step 5.
- Files: `commands/immo/analyze.md`, `agents/immo-analyzer-agent.md`
- Impact: Tax benefit calculations for years 5-10 are systematically imprecise. For 100% LTV loans with low Tilgung rates, the error compounds across all shortlisted units and propagates into the advisor briefing.
- Fix approach: Implement an amortization formula for remaining loan balance at year 5 and recalculate annual interest properly; at minimum, document the approximation as an estimate.

**No Schema Validation on config.json at Runtime:**
- Issue: `immo/templates/config.json` declares `"$schema": "https://immo.dev/schemas/config.schema.json"` but the domain `immo.dev` does not host a real schema (no evidence of a hosted schema). Commands that read `.immo/config.json` (analyze, filter, compare, stress-test, report) do so with `Read` and parse fields by reference — there is no validation step.
- Files: `immo/templates/config.json`, `commands/immo/init.md`, `commands/immo/analyze.md`
- Impact: Malformed configs (wrong types, missing fields) produce silent downstream errors in calculations. A user who manually edits `.immo/config.json` with an incorrect `marginalRate` value gets no warning.
- Fix approach: Either host the JSON schema at the declared URL, or add a Phase 1 validation step in `analyze.md` and `filter.md` that checks required fields and types before proceeding.

---

## Security Considerations

**Sensitive Financial Data Written to Plain Text:**
- Risk: `.immo/config.json` stores investor income, tax rate, liquid assets, and monthly commitments in clear text on disk. The config is read by all commands and referenced in generated reports.
- Files: `immo/templates/config.json`, `commands/immo/init.md`, `commands/immo/report.md`
- Current mitigation: `.gitignore` is present but its contents were not verified to exclude `.immo/` directory. No encryption or access controls exist.
- Recommendations: Verify `.immo/` is in `.gitignore`; document to users that the folder contains sensitive financial data; warn users not to commit the `.immo/` directory to version-controlled project repos.

**`bin/install.js` Silent Directory Deletion:**
- Risk: The `install` function in `bin/install.js` calls `fs.rmSync(destDir, { recursive: true })` before copying files (line 78-79). On global install (`~/.claude/commands/immo/`, `~/.claude/immo/`), this destroys any pre-existing user customizations without confirmation.
- Files: `bin/install.js` lines 77-79
- Current mitigation: None. The uninstall path checks existence before deleting, but the install path does not prompt the user.
- Recommendations: Add a backup step or prompt before destructive removal; at minimum, log a warning that existing files will be overwritten.

**PATH Replacement Regex Is Too Broad:**
- Risk: `bin/install.js` lines 92-93 replace `~/\.claude\/` globally in all `.md` files. If any command file legitimately contains the string `~/.claude/` as documentation or an example, it will be silently rewritten to the install path prefix.
- Files: `bin/install.js` (function `copyWithPathReplacement`)
- Current mitigation: None.
- Recommendations: Use a more targeted replacement (e.g., only replace at line start or within specific YAML frontmatter fields); add a test that verifies substitution correctness.

---

## Performance Bottlenecks

**`/immo:report` Loads All Research Files Unconditionally:**
- Problem: `commands/immo/report.md` Phase 1 reads all `.immo/research/locations/*.md`, all `.immo/analysis/*/SHORTLIST.md`, and the latest `RATES-*.md` in sequence before generating the report. There is no caching or incremental loading.
- Files: `commands/immo/report.md`
- Cause: All data loading happens in one blocking phase; as the number of analyzed locations grows, startup time grows linearly.
- Improvement path: Low priority for typical use (few locations per project), but add a "load only shortlisted locations" filter to avoid reading EXCLUDED research files.

**PDF Generation Falls Back Silently Through Three Tools:**
- Problem: `commands/immo/report.md` Phase 8 tries pandoc → mdpdf → md-to-pdf via `command -v` and `npx --yes` invocations. The `npx --yes md-to-pdf` call downloads the package on first use, adding latency with no progress indicator to the user.
- Files: `commands/immo/report.md` Phase 8
- Cause: No preferred tool is pre-installed; discovery is runtime-only.
- Improvement path: Add a pre-flight check in `bin/install.js` or document a one-time setup step; surface download happening to the user.

---

## Fragile Areas

**`/immo:update` Version Detection is Brittle:**
- Files: `commands/immo/update.md`
- Why fragile: Step 1 parses `npm list -g immo-cc --depth=0` output with `sed 's/.*@//'`. If npm changes its output format, the version string breaks silently. Step 1 also checks `~/.claude/immo/package.json` which is not written by `bin/install.js` (only `VERSION` file is written, not `package.json`), so the local package.json check will always fail.
- Safe modification: Replace with `npm view immo-cc version` for the installed version check, or rely solely on the `VERSION` file that is actually written.
- Test coverage: None.

**Excel Extraction Is Fully Delegated to Claude Without Validation:**
- Files: `commands/immo/analyze.md` Phase 3, `agents/immo-analyzer-agent.md` Steps 1-2
- Why fragile: The system relies on the AI to interpret Excel column headers (WE, Einheit, Wohnfläche, Kaltmiete, etc.) from arbitrary developer files. Column names vary by developer and language. Misidentified columns (e.g., confusing Kaufpreis with a formula cell) propagate silently into all downstream calculations.
- Safe modification: Add an explicit validation step that cross-checks extracted unit count against expected rows, and flags any unit where `price == 0` or `size == 0`.
- Test coverage: None.

**STATE.md Format Is Defined Inline in Multiple Commands:**
- Files: `commands/immo/analyze.md` Phase 7, `commands/immo/scout.md` Phase 9, `commands/immo/filter.md` Phase 6, `commands/immo/compare.md` Phase 9, `commands/immo/stress-test.md` Phase 8, `commands/immo/report.md` Phase 9
- Why fragile: Each command independently defines what STATE.md should look like when it updates a row. There is no shared schema. If one command writes `📊 ANALYZED` and another expects `ANALYZED` (no emoji) when reading, parsing breaks silently.
- Safe modification: Centralize STATE.md format in `immo/references/methodology.md` or a dedicated `immo/references/state-schema.md`; have all commands reference it.
- Test coverage: None.

**`/immo:filter` References `/immo:set` Command That Does Not Exist:**
- Files: `commands/immo/filter.md` edge case section ("No Units Qualify")
- Why fragile: The "No Units Qualify" path suggests the user run `/immo:set minYield [new-value]` to relax criteria — but no `set.md` command exists in `commands/immo/`.
- Safe modification: Either add `commands/immo/set.md` or change the suggestion to instruct the user to manually edit `.immo/config.json`.
- Test coverage: None.

**`/immo:compare` Does Not Write a File:**
- Files: `commands/immo/compare.md`
- Why fragile: The compare command outputs comparison tables to the conversation only and updates `STATE.md`. It does not save a comparison file to `.immo/`. If the session ends, the comparison is lost and must be regenerated. The `/immo:report` command reads from `SHORTLIST.md` files directly, not a persisted comparison.
- Safe modification: Add a Phase to write `.immo/analysis/COMPARISON-[DATE].md` for persistence and reproducibility.
- Test coverage: None.

---

## Missing Critical Features

**No Multi-Country Tax Support:**
- Problem: `init.md` presents Germany, Portugal, Spain, and Other as options, but only Germany has tax rules implemented.
- Blocks: International investor use cases; any non-German analysis runs on incomplete logic with no explicit error.

**No `/immo:set` Command:**
- Problem: Referenced in `commands/immo/filter.md` but not implemented.
- Blocks: In-place criteria adjustment workflow; users must manually edit JSON.

**No `/immo:exclude` Command:**
- Problem: Referenced in `commands/immo/filter.md` ("Manual Exclusions: If user has manually excluded units via `/immo:exclude`...") but no `exclude.md` exists.
- Blocks: The documented workflow for manual unit exclusion.

**No Test Suite:**
- Problem: No test files, no test runner config, no `scripts/` contents. Financial calculations (yield, tax benefit, ROE, amortization approximation) have no automated verification.
- Blocks: Confident refactoring of calculation logic; detection of regressions if formulas change.
- Risk: Calculation errors in `analyze.md` or `agents/immo-analyzer-agent.md` propagate into advisor briefings without any safety net.

**VERSION File Not Checked During Analysis:**
- Problem: `bin/install.js` writes a `VERSION` file to `~/.claude/immo/VERSION`. No command reads this to detect stale installations or prompt updates before analysis runs.
- Blocks: Users may run analysis with outdated reference docs (tax rules, methodology) without knowing.

---

## Test Coverage Gaps

**All Calculation Logic:**
- What's not tested: Gross yield, after-tax cashflow Y1-4/Y5-10, upfront costs, ROE, exit value, tax benefit calculations.
- Files: `agents/immo-analyzer-agent.md`, `commands/immo/analyze.md`
- Risk: Any formula error goes undetected until a user notices the output looks wrong.
- Priority: High

**Install/Uninstall Script:**
- What's not tested: `bin/install.js` path replacement, directory creation, file copying, global vs local install, uninstall idempotency.
- Files: `bin/install.js`
- Risk: Silent regressions on path handling across OS environments (macOS vs Linux).
- Priority: Medium

**Filter Logic:**
- What's not tested: Criteria application, edge cases (0 qualifying units, near-misses), multi-criteria exclusion order.
- Files: `commands/immo/filter.md`
- Risk: Units may be incorrectly excluded or included based on criteria edge cases.
- Priority: Medium

---

*Concerns audit: 2026-04-06*
