---
name: acceptance-testing
description: >
  Functional acceptance testing with requirements traceability. Loaded after
  Phase 7 completes. Traces Phase 1 requirements through Phase 2 components,
  Phase 3 test cases (Phase 4 test code verified through Phase 6 execution results and
  direct file existence checks), to Phase 5 implementation, with Phase 6-7 evidence.
  AC-level verification with evidence hierarchy, independent check, and
  release decision.
---

# Acceptance Testing — Phase 8: Functional Verification

**Core Principle**: Build the right thing, not just build things right. Phase 6/7 verified correctness. Phase 8 verifies completeness.

**Scope Boundary**: Phase 8 diagnoses coverage gaps and reports findings to the user for decision. It does NOT initiate upstream phase re-runs, prescribe rollback targets, or define quality standards for upstream deliverables.

**When loaded**: After Phase 7 verification audit passes.

**Edge cases at load**:
- **Zero ACs**: If Phase 1 produced zero Acceptance Criteria, skip Parts 1–3 and proceed directly to Part 5 with an empty acceptance report (if C/H findings exist, skip Part 4 and report directly — see Category 3 handling). The empty report contains: Coverage Summary ("Core ACs: 0/0 release-ready" and "Secondary ACs: 0/0 release-ready"), plus Phase 7 Impact (system-level findings only, if any) and Phase 6 System-Level Checks (if any system-level results exist). All AC-dependent sections (Traceability Matrix, Blockers, Warnings, Non-Automatable Core ACs) are omitted. **Phase 7 unresolved C/H and Phase 6 system-level FAILs are BLOCKERs regardless of AC count** — they must be checked even with zero ACs. Report Status = original.
- **Zero core ACs (only secondary)**: Run Parts 1–4 normally; the Coverage Summary will show "Core ACs: 0/0 release-ready" and the Blockers table will have no core-AC BLOCKER rows (but Phase 7 C/H and Phase 6 system-level BLOCKERs, if any, still appear as separate rows — they are independent of AC count), secondary warnings still apply.
- **Normal case**: Proceed with Parts 1–5 as documented.

**Execution**: Parts 1–3 (matrix construction, coverage verification, report) are executed by the main agent or a dedicated acceptance subagent. Part 4 must be performed by a different agent to ensure independence. **Load dependencies**: Phase 8 references `phase-7-system-quality-audit.md` for Phase 7 C/H handling (see Rollback Guidance section in that file) — ensure this document is available in context. For split (task-tree) projects: merge per-module Phase 3 coverage matrices into a unified traceability matrix before diagnosis; all AC→Component→Test mappings span modules — use Phase 2's integrated Component Table as the cross-module reference. **AC renumbering after merge**: if modules produce duplicate AC identifiers (e.g., both have "AC-3"), renumber sequentially across the unified matrix in the order modules are listed in Phase 2's component map (AC-3a→AC-3, AC-3b→AC-4, etc.) to ensure unique suffixes for threshold ordering. **Merge conflict resolution**: if modules provide conflicting data for the same AC (e.g., different component mappings or evidence levels), use the module whose Phase 2 Component Table owns the AC's primary component; if ownership is ambiguous, flag for user resolution before proceeding. **Non-conflicting supplementary data**: if the same AC appears in multiple modules with non-conflicting data (e.g., each module contributes different test cases), union all test cases and evidence — do not arbitrarily choose one module's data.

---

## Part 1: Traceability Matrix Construction

### Data Sources

| Source | Extract |
|--------|---------|
| Phase 1 | All User Stories + Acceptance Criteria (core/secondary) |
| Phase 2 | AC→Component mapping (extract from the "Serves Phase 1 ACs" column of Phase 2's Component Table; bidirectional — also used for component→AC cross-validation) |
| Phase 3 | Requirements Coverage Matrix (AC→Test Case mapping) + Design Coverage Matrix (Component→Test Case) |
| Phase 6 | Sub-Phase pass records (test case name → pass/fail) + Release Gate Checklist (evidence compilation for release decision) + re-run metadata (if Phase 6 was re-run after a Phase 7 fix, the results must include a `re-run: true` indicator or equivalent timestamp — Phase 8 uses this to distinguish "never re-run" from "re-run produced no evidence" for Escalated Unknown classification) |
| Phase 7 | Audit report with findings (fixed/rejected items serve as implementation evidence) + unresolved findings list (should be empty after Phase 7 gate; defensive check for C/H) |

Note: Phase 4 (Test Code) and Phase 5 (Business Code) outputs are not listed as separate data source rows in the Data Sources table — their coverage is captured indirectly through Phase 6 test results (which run Phase 4 tests) and source code (which contains Phase 5 implementation). However, Part 2's diagnosis evaluation may need to check Phase 4 test file existence and Phase 5 code file existence directly to distinguish between No Test Code, No Impl, Weak, and Unknown diagnoses (see Diagnosis Table conditions). Part 4 (Independent Verification) receives test code files and implementation source code directly for spot-checking.

### Merge Rules

From Phase 3's Requirements Coverage Matrix, extract `Test File` and `Test Name` — merge into single `Test Case(s)` column: `test_file::test_name`. Phase 3's template uses bare filenames (e.g., `test_email.py`); normalize to filenames only (strip directory paths if present). **"—" in Test Case(s) column**: means no named test case exists for this component/AC — used for manual-only evidence (L3) or complete absence of test coverage. When an AC has some components with test cases and others without, use the per-component format: `test_case (evidence_level), — (evidence_level)`.

Phase 3's Design Coverage Matrix provides component→test cross-validation: for each AC row, use Phase 2's AC→Component mapping to find components, then cross-check Phase 3 Design Coverage Matrix (filter for rows where `Element Type` = Component): the component should have ≥1 test case listed. If not, flag as potential coverage gap → list the flagged component(s) and their parent AC(s) in the Part 3 report under Warnings (with "Cross-validation gap: no test case for component" in Notes). These flagged gaps do not trigger rollback but must be reviewed by the Part 4 independent verifier. (Design Coverage Matrix `Design Element` column where `Element Type` = Component = Component Table `Component name` from Phase 2.)

Phase 6 Sub-Phase 0 output (pytest/vitest) maps test case names to pass/fail/skip status. Only **pass** results count as L1 evidence; a FAIL or SKIP result means no L1 evidence for that test case. (SKIP results from scaffold tests with `skip` markers are expected for non-automatable ACs and do not affect Evidence Level.) (Phase 6 internally uses "Phase 0/1/2/3"; this document uses "Sub-Phase 0/1/2/3" to avoid confusion with pipeline Phases 1–8 — same entities, different naming convention. Sub-phases may be further subdivided: 1.5, 1.6, etc. represent intermediate checkpoints within their parent Sub-Phase.) If Phase 6 results are not in structured format, report affected ACs as Unknown (cannot verify evidence format) — the user may direct a Phase 6 re-run with structured output flags (`--tb=no -q` for pytest, `--reporter=verbose` for vitest). Phase 8 does not initiate re-runs.

**For other test frameworks** (Jest, Mocha, JUnit, go test, cargo test, etc.): extract test file and case name from verbose/detailed output, then normalize to the same `test_file::test_name` format. **Format normalization**: strip only the final file extension from `test_file` (e.g., `test_email.py::should_accept` → `test_email::should_accept`; `user.test.ts::login` → `user.test::login` — keep `.test`/`.spec` as part of the base name). Strip directory paths if present (Phase 3 uses bare filenames). For vitest, verbose output uses `✓`/`×` prefixes with file paths — extract test file and name, then normalize to the same `test_file::test_name` format.

For parameterized tests, use the base test name (before `[`) for AC mapping, preserve the full name in evidence. If a single parameterized test covers multiple ACs (different parameters validate different requirements), list all covered ACs in the matrix row's Notes column and preserve the parameter→AC mapping in the report.

Sub-Phase 1 integration/E2E results provide L2 evidence — extract similarly. When an integration/E2E test covers multiple ACs, assign L2 evidence to all ACs whose requirements the test validates; document the mapping rationale in the report. When an integration/E2E test partially validates an AC (e.g., validates API layer but not UI layer of a multi-component AC), record L2 for the validated portion and note the partial coverage in the report — do not grant full L2 to the AC unless all its components are covered.

Sub-Phase 3 manual exploratory checklist results provide L3 evidence. Map each checklist item to the relevant AC row via the component(s) column. If a manual check validates an AC with no automated test, set Evidence Level to L3.

Phase 7 fixed issues: if Phase 7 found and fixed bugs, the fixes are new implementation evidence. If Phase 6 re-run results (performed after the Phase 7 fix) are available AND show L1/L2, upgrade Evidence Level for the relevant AC rows. If no re-run results are available, retain the original Evidence Level and note the fix in the Phase 7 Impact section (do not upgrade).

### Matrix Format

```
| AC# | US# | Priority | AC Description | Component(s) | Test Case(s) | Evidence Level | Status | Notes |
|-----|-----|----------|---------------|--------------|--------------|----------------|--------|-------|
| AC-1 | US-1 | core | Given X When Y Then Z | EmailValidator | test_email::should_accept, test_email::should_reject | L1 | ✅ | |
| AC-2 | US-1 | core | Given X When Y Then Z | UserService | — | — | ❌ | No Test Plan (Phase 3 has no row) |
| AC-3 | US-2 | secondary | Given X When Y Then Z | ProfileUI | — | L3 | ⚠️ | Weak (manual-only evidence) |
| AC-4 | US-3 | secondary | Given X When Y Then Z | SearchUI | — | — | ❌* | |
| AC-5 | US-3 | core | Given X When Y Then Z | PaymentService | test_payment::should_timeout | — | ⚠️† | Weak→Unknown: test code exists |
| AC-6 | US-4 | core | Given X When Y Then Z | OrderAPI, OrderDB | test_order::should_create (L1), — (L3) | L3 | ⚠️ | Multi-component weakest-link |
```

\* Secondary ❌ = warning (not a BLOCKER). See Secondary AC Rules.
† Unknown diagnosis (implementation + test code exist but no Phase 6 evidence). Distinguish from Weak (⚠️ with L3) by checking Evidence Level column.
‡ AC-6 multi-component weakest-link: OrderAPI diagnosed Covered (test + L1 evidence), OrderDB diagnosed Weak (no L1/L2, only L3 evidence). Weakest-link: Weak > Covered → overall diagnosis = Weak. Overall Evidence Level = min(L1, L3) = L3.

> **Note on example matrix**: This matrix shows the state after Part 2 diagnosis. The Secondary Threshold Check runs after all ACs are diagnosed — it converts qualifying secondary ⚠️ to ✱ in the matrix Status column (threshold calculation: floor(20% × total_secondary_ACs)). In this example with only 2 secondary ACs, floor(20% × 2) = 0, so no secondary ACs qualify for ✱. In a larger project (e.g., 10 secondary ACs with 3 Weak), the top 2 Weak secondaries would show ✱ after threshold check.

> **Status symbols**: ✅ = Covered, ❌ = gap (BLOCKER for core, warning for secondary — check Priority column), ⚠️ = ambiguous — check Evidence Level column to distinguish: (1) Weak = ⚠️ with L3 evidence, (2) Non-automatable core = ⚠️ with L3 + justification in Non-Automatable Core ACs section. Non-automatable core Weak ACs are not BLOCKERs. ⚠️† = Unknown (Weak→Unknown reclassification via sub-dispatch; always appears as ⚠️† in the matrix because Weak catches "—" cases first via first-matching-wins, then sub-dispatch reclassifies — the Diagnosis Table's Unknown row defines the condition and action, but the Weak sub-dispatch is the actual entry path). ✱ = accepted secondary Weak (⚠️ with L3, within 20% threshold — not a blocker, not a warning, deliberately accepted manual-only coverage). **Note**: The Diagnosis Table uses ❓ (Bad Requirement), ❌ (gap diagnoses), ⚠️ (Weak/Unknown), and ✅ (Covered) as visual prefixes — these match the Status column symbols except ❓, which maps to ❌ in the matrix (Bad Requirement is a gap diagnosis). Plain ⚠️ with "—" evidence (listed in older versions as "Unknown") should never appear in a correctly constructed matrix — use ⚠️† instead.

### Evidence Levels

**Ranking** (highest to lowest): L1 > L2 > L3 > — (none). L4 is excluded from this ranking because it is report-level metadata (see L4 row below), not per-AC test evidence. **Within a single component**: when multiple evidence sources exist for the same component (e.g., L1 from unit test + L2 from integration test), use the highest level — any L1 or L2 evidence means that component is "Covered" regardless of L3 or L4 status. **Across components**: for multi-component ACs, use the weakest-link rule (see Multi-AC section below) — the AC's overall Evidence Level is the lowest across its components, which may override Covered status.

| Level | Source |
|-------|--------|
| L1 | Phase 6 Sub-Phase 0 (unit test pass) |
| L2 | Phase 6 Sub-Phase 1 (integration/E2E pass) |
| L3 | Phase 6 Sub-Phase 3 (manual exploratory checklist results) |
| L4 | Part 4 independent verification (recorded in Report Status field, not in Evidence Level column — L4 is metadata about the report's accuracy, not per-AC test evidence) |

> **"—" (dash) in Evidence Level column**: means "no evidence found" — used for two cases that are distinguishable by the Status column: (1) gap diagnoses (❌ No Test Code, ❌ No Impl, etc.) where "—" indicates the absence that defines the diagnosis; (2) Unknown diagnosis (⚠️† with "—") where the AC has implementation + test code but Phase 6 produced no evidence record.

> **Why L4 (highest-numbered) is "Weak" in isolation**: L4 confirms the *report's accuracy* (metadata about evidence), not the AC's *test automation depth*. L4 alone means "the verifier says the report is correct, but the AC has no automated test coverage." Automated test evidence (L1/L2) is preferred because it is repeatable, objective, and regression-aware. Manual/L4-only evidence is "Weak" because it cannot be re-executed cheaply and may degrade over time. An AC with L1 or L2 evidence is automatically "Covered" regardless of L4 status.
>
> **L4 write mechanism**: L4 is NOT set during Parts 1–3 (matrix construction) — it cannot exist before Part 4 runs. Instead: (1) Parts 1–3 assign L1/L2/L3 to matrix rows based on Phase 6/7 evidence; (2) Part 4 verifies the report; (3) Part 4's outcome is recorded in the Report Status field (`original` or `corrected (N factual errors fixed)`) — this is the L4 evidence. L4 does NOT need a separate row in the Evidence Level column; it is captured through the Report Status field.

Note: Phase 6 Sub-Phase 1.5 (Soak), 1.6 (Contract), and 2 (Cross-Cutting) are system-level checks, not per-AC evidence — they don't map to individual AC rows. Sub-Phase 2 results are excluded for the same reason as 1.5/1.6: they validate system-wide properties (config consistency, CI reproducibility, regression guard) rather than specific AC compliance. Results from these system-level checks are captured in the Phase 6 Release Gate Checklist and reported in the Phase 6 System-Level Checks section of the Part 3 report (see Part 3). A failing system-level check may indicate a broader quality issue that affects multiple ACs even though it does not appear in any single AC's evidence row.

---

## Part 2: Coverage Verification

For each AC in the traceability matrix, diagnose its coverage status. **Unknown diagnosis (core)**: report as BLOCKER — the user decides whether to trigger a Phase 6 re-run. Do NOT initiate re-runs from Phase 8. For secondary Unknowns, record as warning.

### Diagnosis Table

Evaluate in order — first matching diagnosis wins:

| Diagnosis | Condition | Verdict |
|-----------|-----------|---------|
| ❓ Bad Requirement | Requirement is infeasible or self-contradictory | core→BLOCKER; secondary→warning |
| ❌ No Design | Phase 2 has no component mapping for this AC | core→BLOCKER; secondary→warning |
| ❌ No Test Plan | Phase 3 Requirements Coverage Matrix has no row for this AC | core→BLOCKER; secondary→warning |
| ❌ No Test Code | Phase 3 has a row but any corresponding test file listed in that row is absent from the project's test directories (check paths from Phase 3's test plan; if Phase 3 did not specify paths, scan `test/`, `tests/`, `__tests__/`, and `spec/`), OR test file exists but the specific test function named in Phase 3's row is absent from the file. For multi-component ACs where Phase 3 lists multiple test files (one per component), No Test Code fires if ANY listed file is missing | core→BLOCKER; secondary→warning |
| ❌ No Impl | Phase 2 has component mapping but no corresponding source code file outside the `test/` or `tests/` directories (implementation files reside in `src/`, `lib/`, `app/`, or other non-test directories per project structure) | core→BLOCKER; secondary→warning |
| ⚠️ Weak | implementation exists but no L1/L2 evidence (Evidence Level = L3 or "—") | core→BLOCKER (unless inherently non-automatable, document justification); secondary→accepted up to 20% (see Secondary Threshold Check); excess→warning |
| ⚠️ Unknown | (reached via Weak→Unknown reclassification only — see below) | core→BLOCKER (report to user; user decides whether to re-run Phase 6); secondary→warning (recorded in report) |
| ✅ Covered | test exists + L1 or L2 evidence (Phase 6 Sub-Phase 0/1 pass) + implementation exists | PASS |

> **Escalated Unknown**: An Unknown-diagnosis AC becomes "Escalated" when Phase 6 was re-run after the Unknown diagnosis and evidence is still missing (this implies Phase 8 re-invocation after user-directed Phase 6 re-run — see Part 5 Step 1). Escalated Unknowns are presented to the user with three options (Part 5 Step 1): accept risk, manual investigation, or shelve. Escalation context is recorded in the Accepted Risks table.

> **Test-to-AC correctness**: The "Covered" diagnosis verifies that a test exists and passes. It does NOT verify that the passing test correctly validates the AC's requirements. Part 4 (Independent Verification) spot-checks this by reading test code and confirming it exercises the AC's "Given/When/Then" scenario. If Part 4 finds a test that passes but tests the wrong behavior, it records this as a factual error (not a BLOCKER in Part 2). **Re-diagnosis after factual error correction**: when a Covered AC's test is found to test wrong behavior, the corrected report must re-evaluate the affected AC against the Diagnosis Table from Bad Requirement downward. The test still exists and passes, so the AC remains Covered — but the Part 4 report records that the test's validity was verified and any corrections applied. If the correction invalidates the test entirely (e.g., test was testing a different AC's scenario — meaning the test's "Given/When/Then" maps to a different AC's requirements, not merely testing the right behavior in a wrong way), remove the test case mapping and re-diagnose; the new diagnosis replaces the old one in the matrix and Blockers/Warnings tables.

### Weak→Unknown Reclassification (sub-dispatch)

After Weak matches via first-matching-wins, apply this sub-dispatch to check if the diagnosis should be reclassified from Weak to Unknown (reduced severity per severity order):

**Condition**: Evidence Level = "—" (zero Phase 6 evidence, not even L3) AND automated test code already exists for this AC (Phase 3 lists a test case AND the corresponding test file from Phase 4 exists with the named test function).

**Action**: Reclassify to Unknown. Update Status from ⚠️ to ⚠️†. Record Unknown (not Weak) in the Blockers table.

**Root cause**: Phase 6 did not run/capture the test — the gap is in evidence collection, not in test coverage.

**Does NOT fire when**: Evidence Level = L3 (Phase 6 did run and produced manual evidence, just no L1/L2 — Weak is genuine).

**Core-only root cause exception**: If Phase 3 planned only manual testing for this AC (not automated), root cause is Phase 3 — report accordingly.

**Multi-component ACs**: When an AC maps to multiple components (per Phase 2's AC→Component mapping), apply the **weakest-link rule**: the AC's overall diagnosis equals the worst diagnosis among its components. **Severity order** (worst to best): Bad Requirement > No Design > No Test Plan > No Test Code > No Impl > Weak > Unknown > Covered (matches Diagnosis Table top-to-bottom order).

**Per-AC vs per-component diagnosis scope**: Diagnoses partition into two groups:
- **Per-AC diagnoses** (Bad Requirement, No Design, No Test Plan, No Test Code): evaluated once for the entire AC using Phase 1–3 artifacts. These do not differ across components. If a per-AC diagnosis matches, it applies to the whole AC regardless of individual component states.
- **Per-component diagnoses** (No Impl, Weak, Unknown, Covered): evaluated independently per component. No Impl checks whether each component has a code file. Weak checks whether each component has L1/L2 evidence. Unknown checks whether each component has Phase 6 evidence. Covered confirms each component has a test that passes with L1/L2 evidence.

Apply weakest-link across per-component diagnoses only. If any per-AC diagnosis matches (Bad Requirement through No Test Code), it takes precedence over all per-component diagnoses — the AC-level gap makes component-level evaluation moot. If no per-AC diagnosis matches, evaluate each component for No Impl / Weak / Unknown / Covered independently, then apply weakest-link.

Evidence Level uses the lowest evidence level across components (e.g., Component A = L1, Component B = L3 → AC Evidence Level = L3, diagnosis = Weak). List all components' evidence in the Test Case(s) column using the format `test_case (evidence_level)` per component, comma-separated (e.g., `test_order::should_create (L1), — (L3)`) so the mixed coverage is visible. **Scope note**: "first matching wins" applies within a single component's diagnosis evaluation; "weakest-link" applies across components of the same AC. These are different scopes, not conflicting rules.

**Missing requirement detection**: Use Phase 2's AC→Component mapping to identify which components serve each AC, then locate those components' code files through the project's source directory structure (Phase 5 output). Do not keyword-grep Phase 5 code for requirement text.

> **Pipeline integrity assumptions**: The diagnosis table assumes Phase 6 gate passed (all tests passing) and Phase 7 gate passed (no unresolved C/H findings). A "test exists + implementation exists + Phase 6 FAIL" scenario indicates a pipeline violation (Phase 6 gate bypassed or Phase 7 fix without re-run). Report as Unknown diagnosis with a note about the pipeline violation. Unresolved C/H from Phase 7 should not occur (defensive check in Phase 7 findings table, category 3).

**Unresolved Phase 7 findings** — Phase 7 findings are handled in four categories:

| Category | Action | Risk Assessment |
|----------|--------|-----------------|
| (1) Fixed findings | Reflected as implementation evidence in relevant AC rows (see Phase 7 fixed issues rule above) | N/A |
| (2) Unresolved M/L findings | Listed in Phase 7 Impact section with full risk assessment | Required: finding ID, potential impact on evidence validity, recommended user attention level |
| (3) Unresolved C/H findings | Treated as BLOCKER (defensive check: should not occur after Phase 7 gate). Do not complete Parts 3–5; report to user with an abbreviated report: list of unresolved C/H findings, their categories, and a statement that full acceptance assessment is deferred pending resolution. | N/A (BLOCKER) |
| (4) Unresolved P findings | Noted in Phase 7 Impact section for completeness only | Not required (informational) |
| (5) Rejected findings | Noted in Phase 7 Impact section as informational context (rejected items are already reflected as implementation evidence per L38) | Not required (informational — rejection confirms the original behavior was acceptable) |

> **Why M gets high attention, P gets medium, and L gets low**: Pipeline severity ranks P (Petty) above L (Low), but attention level reflects required user action, not severity rank. M findings may indicate latent defects — review before release decision. P findings are cosmetic (formatting/wording) with no functional risk, but acknowledging them confirms awareness. L findings are purely informational observational insights — no user action required for the release decision, recorded for completeness only. **Note**: "no action required" refers to the user's release-decision response; the agent must still complete all risk assessment fields (finding ID, potential impact, attention level) for M/L findings in the report. The grouping follows action-necessity, not severity rank order.

**User attention levels** for unresolved findings below C/H severity (defined here for Phase 8 report presentation; Phase 7 severity classification is the source of truth for finding severity):

| Phase 7 Severity | Phase 8 Attention Level | Meaning |
|-----------------|------------------------|---------|
| M (Medium) | **high** | Review before release decision — may indicate a latent defect or incomplete fix |
| P (Petty) | **medium** | Acknowledge awareness — cosmetic/formatting concern, no functional risk |
| L (Low) | **low** | Informational, no action required — observational insight only |

**Phase 7 findings → AC mapping**: For each Phase 7 finding, identify affected components via file:line references, then use Phase 2's AC→Component mapping to find associated ACs. Findings whose affected components are not in Phase 2's Component Table, or that have no specific component reference, are system-level and noted separately in the Phase 7 Impact section.

**Secondary Threshold Check**: After diagnosing all ACs, count secondary ACs with `Weak` diagnosis. If this count exceeds 20% of ALL secondary ACs in the project, list the excess in the Warnings section of the report. **Selection for accepted vs excess**: among secondary Weak ACs, the first N (where N = floor(20% × total_secondary_ACs)) are accepted (✱), ordered by their AC# numeric suffix (lowest first — extract the numeric portion of AC-1, AC-2, AC-10 etc. and sort numerically, not lexicographically); remaining are excess and listed in Warnings. (Edge case: if the project has zero secondary ACs, this check is vacuous — skip it.)

### Core AC Rules

⛔ `Bad Requirement` → BLOCKER
⛔ `No Design` / `No Test Plan` / `No Test Code` / `No Impl` → BLOCKER
⛔ `Weak` (no L1/L2; Evidence Level = L3 — after Weak→Unknown reclassification, all remaining Weak cases have L3 evidence, never "—") → BLOCKER, unless the AC is inherently non-automatable (document justification in report). **Note**: "Non-automatable" means L1/L2 automated assertion is infeasible, but Phase 4 test code is still expected (the AC's absence of L1/L2 is justified, not the absence of test code). Absence of test code is diagnosed as No Test Code regardless of automatability. **Expected test code form by justification category**: (1) visual/appearance → scaffold test with `skip` marker and manual procedure comments; (2) hardware-specific → scaffold test with mock interfaces and hardware condition guards; (3) third-party service → scaffold test with recorded fixtures or contract stubs.
⛔ `Unknown` (implementation + test code exist but no Phase 6 evidence) → BLOCKER (report to user; user decides resolution)
✅ `Covered` (L1 or L2) → PASS

### Secondary AC Rules

`No Test Plan` / `No Test Code` / `No Impl` / `No Design` → warning (recorded in report, user decides)
`Bad Requirement` → warning (recorded in report, user decides whether to rollback to Phase 1)
`Unknown` → warning (recorded in report)
`Weak` → max 20% of secondary ACs may have only L3 evidence post sub-dispatch (threshold calculated as floor(20% × total_secondary_ACs); the hard gate is that all excess above this exact count must be listed as warnings — the threshold does not block release but ensures visibility); excess → warning
✅ `Covered` → PASS

---

## Part 3: Acceptance Report

### Format

```markdown
# Acceptance Report

**Report Status**: original | corrected (N factual errors fixed)

## Coverage Summary
- Core ACs: X out of Y release-ready (Z blockers, W non-automatable with justification)
- Secondary ACs: X out of Y release-ready, W accepted manual-only (Z warnings)

> **"Release-ready" definition**: Status = ✅ (Covered) for both core and secondary, ✱ (accepted secondary Weak — secondary ACs only, never applies to core), or ⚠️ (non-automatable core with justification — core ACs only). ❌ (gap) and ⚠️† (Unknown) are not release-ready for either priority. **Secondary ❌ ACs**: secondary gap diagnoses (No Test Code, No Impl, etc.) are warnings, not BLOCKERs — they are counted in the Coverage Summary's "Z warnings" count but reduce X in "X out of Y release-ready." This differs from the Diagnosis Table's "Covered" diagnosis (which requires L1/L2 evidence specifically).

## Traceability Matrix
[full matrix from Part 1]

## Blockers
| Entity | Type | US# | Diagnosis |
|--------|------|-----|-----------|
| [Part 2 diagnosis: AC ID (e.g., AC-3)] | AC | [parent US] | [diagnosis from table] |
| [Phase 7 unresolved C/H: Finding ID (e.g., F-42)] | Phase 7 | system | Phase 7 C/H |
| [Phase 6 system-level FAIL: Sub-Phase + check name (e.g., SP2: Config consistency)] | System | system | Phase 6 system-level |

## Warnings
> **Note**: The Diagnosis column in this table accepts Diagnosis Table entries, "Cross-validation gap" (a Part 1 synthesized category for components with no test case), and "Weak threshold exceeded" (a Part 2 synthesized threshold summary). The latter two are not Diagnosis Table diagnoses.

| AC# | US# | Priority | Diagnosis | Evidence Level | Notes |
|-----|-----|----------|-----------|----------------|-------|
| [secondary AC coverage gaps] |
| [cross-validation gaps: one row per parent AC (if component serves multiple ACs, create one row for each); parent US# in US#, Priority from AC, "Cross-validation gap" in Diagnosis, "—" in Evidence Level, "no test case for component [name]" in Notes] |
| — | — | secondary | Weak threshold exceeded (X/Y secondary ACs, 20% max) | — | Z excess ACs listed below |
| [excess Weak secondary ACs] |

## Non-Automatable Core ACs (if any)
> **Note**: Test Case(s) column omitted from this table — for non-automatable ACs, test code form varies by justification category (see Core AC Rules note). The Traceability Matrix row for each non-automatable AC still contains the Test Case(s) entry; this table provides the justification for the L3-only evidence.

| AC# | Component(s) | Evidence Level | Justification Category | Justification Detail |
|-----|--------------|----------------|----------------------|---------------------|
| [ACs with L3 only (no L1/L2)] | [mapped components] | L3 | [one of: (1) visual/appearance verification requiring human judgment; (2) hardware-specific behavior not reproducible in test environment; (3) third-party service with no sandbox/staging API; or other with explicit justification] | [specific reason why automated testing is infeasible for this AC] |

## Phase 7 Impact (if any)
**Fixed findings reflected as evidence**: [List: Finding ID → Affected AC# → Evidence Level upgrade (if applicable). Note: if Phase 6 re-run was NOT performed after the fix, use "no upgrade (re-run pending)" instead of an Evidence Level upgrade.]

**Unresolved findings**:
| Finding ID | Severity | Affected AC# | Potential Impact | User Attention Level |
|------------|----------|--------------|-----------------|---------------------|
| [unresolved M/L findings — full risk assessment required] |
| [unresolved P findings — informational only, Potential Impact may be N/A; User Attention Level: medium (per Attention Level table)] |

**System-level findings** (no specific AC mapping):
[List: Finding ID → Severity → Description → User Attention Level]

## Phase 6 System-Level Checks
| Sub-Phase | Check | Result | Notes |
|-----------|-------|--------|-------|
| [Sub-Phase 1.5 (Soak) / 1.6 (Contract) / 2 (Cross-Cutting) — use Sub-Phase number, not just check name; Release Gate items inherit Sub-Phase from their evidence source] | [check name] | [PASS/FAIL] | [from Phase 6 Release Gate Checklist; system-level FAIL → BLOCKER; warnings are informational] |

## Accepted Risks (if any — user-accepted escalated Unknowns only)
| AC# | US# | Original Diagnosis | Escalation Context | User Decision | Risk Acknowledgment |
|-----|-----|--------------------|--------------------|---------------|-------------------|
| [ACs where user accepted escalated Unknown via Part 5 Step 1 option (a)] | [parent US] | Unknown (escalated) | [why Phase 6 re-run failed to produce evidence] | Risk accepted | [summary of what evidence is missing and potential impact] |
```

---

## Part 4: Independent Verification

An independent subagent (oracle or dedicated verifier, not involved in Parts 1–3) receives the acceptance report + Phase 1–3 documents (including Phase 2 Component Table for component isolation verification) + Phase 6 test results + Phase 7 audit report + test code files + implementation source code. **Isolation requirement**: the Part 4 agent must NOT receive Parts 1–3 intermediate reasoning, draft matrices, or diagnostic deliberations — only the final artifacts listed above.

### Verification Checklist

- [ ] Matrix completeness: AC count matches Phase 1 (no ACs silently dropped or added)
- [ ] Test existence — BLOCKERs: **Note: Test Case(s) values come from Phase 3's plan (see Merge Rules); for No Test Code, a test name appears because Phase 3 planned it, but Phase 4 never created it.** For BLOCKER-status ACs with non-empty Test Case(s), verify the diagnosis is correct — for gap diagnoses (No Test Code), confirm the listed test file or function does NOT exist in Phase 4's output; for non-gap diagnoses, confirm ALL listed test cases exist (file + function name). For BLOCKER-status ACs with Test Case(s) = "—", verify the diagnosis correctly identifies the gap (e.g., confirm no test file exists for No Test Code diagnosis).
- [ ] Test existence — non-BLOCKERs: For all non-BLOCKER ACs (Covered + Weak + secondary-priority ACs with warning diagnoses), verify ≥ 20% random sample (minimum 5, or all if fewer than 5), sampled with uniform distribution via a seeded PRNG for reproducibility (e.g., hash of AC IDs modulo sample size)
- [ ] Implementation existence: for a random sample of Covered ACs (≥5 or all if fewer), verify ≥1 source file exists for each mapped component
- [ ] Evidence accuracy: L1 claims trace to Phase 6 Sub-Phase 0 records; L2 to Sub-Phase 1; L3 to Sub-Phase 3 manual exploratory checklist results; L4 to Part 4 Report Status field (original or corrected)
- [ ] Classification correctness: core/secondary labels match Phase 1
- [ ] No Test Plan vs No Test Code: diagnosis correctly distinguishes the two
- [ ] Weak diagnosis accuracy: for each AC diagnosed as Weak, verify the Evidence Level is L3 (not "—"). If Evidence Level is "—" and automated test code exists, the AC should be diagnosed as Unknown (⚠️†) per the Weak sub-dispatch rule, not Weak
- [ ] Non-automatable justification: each Core AC claiming L3 only (no L1/L2) has specific, defensible justification (not generic "hard to test"). Common justifications include: (1) visual/appearance verification requiring human judgment; (2) hardware-specific behavior not reproducible in test environment; (3) third-party service with no sandbox/staging API. Other justifications are accepted if explicitly documented
- [ ] Phase 7 fix reflection: if Phase 7 found and fixed bugs, verify (1) the fixes are reflected as implementation evidence in the relevant AC rows, and (2) Evidence Level was upgraded to L1/L2 if Phase 6 re-run was performed after the fix, OR (3) if no Phase 6 re-run was performed, the fix is noted in the Phase 7 Impact section and Evidence Level was retained (not upgraded)
- [ ] Phase 7 Impact completeness: all unresolved findings from the Phase 7 audit report (below C/H severity) are listed with their affected ACs. System-level findings (no specific AC mapping) are listed in the System-level findings section, not in the per-AC findings table. For M/L findings, verify risk assessment includes finding ID, potential impact, and recommended attention level. For P findings, verify they are listed (informational only, no risk assessment required)
- [ ] Secondary threshold accuracy: count of Weak secondary ACs matches report; >20% threshold correctly applied; excess listed in Warnings section
- [ ] Cross-validation gap identification: for each cross-validation gap flagged in Part 1 (no test case for a component), verify the gap is listed in the Warnings table with correct parent AC# and component name
- [ ] Phase 6 System-Level Checks accuracy: results in the Phase 6 System-Level Checks section match Phase 6 Release Gate Checklist (PASS/FAIL status, Sub-Phase, check name). System-level FAIL entries also appear in the Blockers table with correct format.

### Gate

⛔ Independent verification fails → correct the report (factual errors only). No rollback. Re-verify corrected report. **Re-verification scope**: if corrections affect ≤3 AC rows, re-verify only the affected rows plus **dependent entries** — defined as: (a) any Blockers/Warnings entries that reference the same AC# as a corrected row, (b) if any secondary AC's Weak/non-Weak status changed, re-verify the threshold summary row and all excess Weak entries in Warnings. If corrections affect >3 rows or change the Coverage Summary counts, re-verify the full report. Maximum 3 correction-verification cycles; if factual errors persist after 3 cycles, escalate to user for manual review.

---

## Part 5: User Go/No-Go

The main agent (who executed Parts 1–3) receives the verified report from Part 4 and presents it to the user.

Submit to user: acceptance report (Part 3, verified by Part 4, or empty per zero-AC skip rule) + Phase 6 Release Gate Checklist + Phase 7 audit report.

**Step 1: Evaluate BLOCKER state**

| Condition | Action |
|-----------|--------|
| Zero BLOCKERs | Proceed to Step 2 (user Go/No-Go) |
| Any BLOCKER (standard) | Present BLOCKERs with diagnoses to user for decision |
| Escalated Unknown BLOCKER (Phase 6 re-run already attempted, evidence still missing) | Present escalation context to user. User decides: (a) accept risk and proceed to Go/No-Go, (b) manual investigation, or (c) shelve (defer release indefinitely) |

> **Option (b) behavior**: Phase 8 pauses and presents a summary of the Escalated Unknown findings to the user. Phase 8 does not resume until the user provides new evidence or direction. The agent remains available for user queries during investigation.

> **Post-verification update for option (a)**: When user accepts an escalated Unknown, the Part 4-verified report is modified — this is intentional and does not require re-verification by Part 4. The main agent must: (1) remove the AC from the Blockers table, (2) add it to the Accepted Risks table with escalation context and user decision, (3) update Coverage Summary blocker count (Z decreases by 1), (4) update the AC's Status in the Traceability Matrix from ⚠️† to a note referencing Accepted Risks. Report Status remains unchanged (Part 4's domain).

**Step 2: User Go/No-Go** (only when zero BLOCKERs)

Present the verified acceptance report to the user for a release decision. Phase 8 reports the decision outcome; it does not execute release or rollback actions.

| Decision | Phase 8 Reports |
|----------|----------------|
| User Go | User approves release → proceed per project's deployment workflow |
| User No-Go | User declines release → user specifies next action (shelve, return to a specific phase, etc.) |

---

## Next Step

User Go → proceed per project's deployment workflow.
