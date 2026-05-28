---
name: acceptance-testing
description: >
  Functional acceptance testing with requirements traceability. Loaded after
  Phase 7 completes. Traces Phase 1 requirements through Phase 2 components,
  Phase 3 test cases (Phase 4 test code verified indirectly through Phase 6), to Phase 5 implementation, with Phase 6-7 evidence.
  AC-level verification with evidence hierarchy, independent check, and
  release decision.
---

# Acceptance Testing — Phase 8: Functional Verification

**Core Principle**: Build the right thing, not just build things right. Phase 6/7 verified correctness. Phase 8 verifies completeness.

**When loaded**: After Phase 7 verification audit passes. (If Phase 1 produced zero Acceptance Criteria, skip Parts 1–4 and proceed directly to Part 5 with an empty acceptance report. If Phase 1 produced only secondary ACs — zero core — run Parts 1–4 normally; the Coverage Summary will show "Core ACs: 0/0 covered" and the Blockers table will be empty for core, but secondary warnings still apply.)

**Execution**: Parts 1–3 (matrix construction, coverage verification, report) are executed by the main agent or a dedicated acceptance subagent. Part 4 must be performed by a different agent to ensure independence. For split (task-tree) projects: merge per-module Phase 3 coverage matrices into a unified traceability matrix before diagnosis; all AC→Component→Test mappings span modules — use Phase 2's integrated Component Table as the cross-module reference. **Merge conflict resolution**: if modules provide conflicting data for the same AC (e.g., different component mappings or evidence levels), use the module whose Phase 2 Component Table owns the AC's primary component; if ownership is ambiguous, flag for user resolution before proceeding.

---

## Part 1: Traceability Matrix Construction

### Data Sources

| Source | Extract |
|--------|---------|
| Phase 1 | All User Stories + Acceptance Criteria (core/secondary) |
| Phase 2 | AC→Component mapping (extract from the "Serves Phase 1 ACs" column of Phase 2's Component Table; bidirectional — also used for component→AC cross-validation) |
| Phase 3 | Requirements Coverage Matrix (AC→Test Case mapping) + Design Coverage Matrix (Component→Test Case) |
| Phase 6 | Sub-Phase pass records (test case name → pass/fail) + Release Gate Checklist (evidence compilation for release decision) |
| Phase 7 | Audit report with findings (fixed/rejected items serve as implementation evidence) + unresolved findings list (should be empty after Phase 7 gate; defensive check for C/H) |

Note: Phase 4 (Test Code) and Phase 5 (Business Code) outputs are not listed as separate data sources for Parts 1–3 — they are verified indirectly through Phase 6 test results (which run Phase 4 tests) and source code (which contains Phase 5 implementation). Part 4 (Independent Verification) receives test code files and implementation source code directly for spot-checking.

### Merge Rules

From Phase 3's Requirements Coverage Matrix, extract `Test File` and `Test Name` — merge into single `Test Case(s)` column: `test_file::test_name`. Phase 3's template uses bare filenames (e.g., `test_email.py`); normalize to filenames only (strip directory paths if present).

Phase 3's Design Coverage Matrix provides component→test cross-validation: for each AC row, use Phase 2's AC→Component mapping to find components, then cross-check Phase 3 Design Coverage Matrix (filter for rows where `Element Type` = Component): the component should have ≥1 test case listed. If not, flag as potential coverage gap → list the flagged component(s) and their parent AC(s) in the Part 3 report under Warnings (with "Cross-validation gap: no test case for component" in Notes). These flagged gaps do not trigger rollback but must be reviewed by the Part 4 independent verifier. (Design Coverage Matrix `Design Element` column where `Element Type` = Component = Component Table `Component name` from Phase 2.)

Phase 6 Sub-Phase 0 output (pytest/vitest) maps test case names to pass/fail status (L1 evidence). (Phase 6 internally uses "Phase 0/1/2/3"; this document uses "Sub-Phase 0/1/2/3" to avoid confusion with pipeline Phases 1–8 — same entities, different naming convention.) If Phase 6 results are not in structured format, re-run with appropriate output flags (`--tb=no -q` for pytest, `--reporter=verbose` for vitest) to extract per-test-case pass/fail — this is a format extraction re-run (different from a full Phase 6 re-run triggered by Unknown diagnosis).

**For other test frameworks** (Jest, Mocha, JUnit, go test, cargo test, etc.): extract test file and case name from verbose/detailed output, then normalize to the same `test_file::test_name` format. **Format normalization**: strip only the final file extension from `test_file` (e.g., `test_email.py::should_accept` → `test_email::should_accept`; `user.test.ts::login` → `user.test::login` — keep `.test`/`.spec` as part of the base name). Strip directory paths if present (Phase 3 uses bare filenames). For vitest, verbose output uses `✓`/`×` prefixes with file paths — extract test file and name, then normalize to the same `test_file::test_name` format.

For parameterized tests, use the base test name (before `[`) for AC mapping, preserve the full name in evidence. If a single parameterized test covers multiple ACs (different parameters validate different requirements), list all covered ACs in the matrix row's notes and preserve the parameter→AC mapping in the report.

Sub-Phase 1 integration/E2E results provide L2 evidence — extract similarly. When an integration/E2E test covers multiple ACs, assign L2 evidence to all ACs whose requirements the test validates; document the mapping rationale in the report. When an integration/E2E test partially validates an AC (e.g., validates API layer but not UI layer of a multi-component AC), record L2 for the validated portion and note the partial coverage in the report — do not grant full L2 to the AC unless all its components are covered.

Sub-Phase 3 manual exploratory checklist results provide L3 evidence. Map each checklist item to the relevant AC row via the component(s) column. If a manual check validates an AC with no automated test, set Evidence Level to L3.

Phase 7 fixed issues: if Phase 7 found and fixed bugs, the fixes are new implementation evidence. Phase 7 fixes should be accompanied by re-running the affected Phase 6 tests. If re-run results are available AND show L1/L2, upgrade Evidence Level for the relevant AC rows. If re-run was not performed, retain the original Evidence Level and note the fix in the Phase 7 Impact section (do not upgrade).

### Matrix Format

```
| AC# | US# | Priority | AC Description | Component(s) | Test Case(s) | Evidence Level | Status |
|-----|-----|----------|---------------|--------------|--------------|----------------|--------|
| AC-1 | US-1 | core | Given X When Y Then Z | EmailValidator | test_email::should_accept, test_email::should_reject | L1 | ✅ |
| AC-2 | US-1 | core | Given X When Y Then Z | UserService | — | — | ❌ |
| AC-3 | US-2 | secondary | Given X When Y Then Z | ProfileUI | — | L3 | ✱ |
| AC-4 | US-3 | secondary | Given X When Y Then Z | SearchUI | — | — | ❌* |
| AC-5 | US-3 | core | Given X When Y Then Z | PaymentService | test_payment::should_timeout | — | ⚠️† |
| AC-6 | US-4 | core | Given X When Y Then Z | OrderAPI, OrderDB | test_order::should_create (L1), — (L3) | L3 | ⚠️ |
```

\* Secondary ❌ = warning (not a BLOCKER). See Secondary AC Rules.
† Unknown diagnosis (implementation + test code exist but no Phase 6 evidence). Distinguish from Weak (⚠️ with L3) by checking Evidence Level column.
‡ AC-6 multi-component weakest-link: OrderAPI has L1 but OrderDB has only L3 → overall Evidence Level = L3 (lowest), diagnosis = Weak.

> **Status symbols**: ✅ = Covered, ❌ = gap (BLOCKER for core, warning for secondary — check Priority column), ⚠️ = ambiguous — check Evidence Level column to distinguish: (1) Weak = ⚠️ with L3 evidence, (2) Unknown = ⚠️ with "—" evidence (no Phase 6 record), (3) Non-automatable core = ⚠️ with L3 + justification in Non-Automatable Core ACs section. Non-automatable core Weak ACs are not BLOCKERs. ✱ = accepted secondary Weak (⚠️ with L3, within 20% threshold — not a blocker, not a warning, deliberately accepted manual-only coverage).

### Evidence Levels

**Ranking** (highest to lowest): L1 > L2 > L3 > — (none). L4 is excluded from this ranking because it is report-level metadata (see L4 row below), not per-AC test evidence. When an AC has multiple evidence sources (e.g., L1 from unit test + L2 from integration test), use the highest level. An AC with any L1 or L2 evidence is "Covered" regardless of L3 or L4 status.

| Level | Source |
|-------|--------|
| L1 | Phase 6 Sub-Phase 0 (unit test pass) |
| L2 | Phase 6 Sub-Phase 1 (integration/E2E pass) |
| L3 | Phase 6 Sub-Phase 3 (manual exploratory checklist results) |
| L4 | Part 4 independent verification (recorded in Report Status field, not in Evidence Level column — L4 is metadata about the report's accuracy, not per-AC test evidence) |

> **"—" (dash) in Evidence Level column**: means "no evidence found" — used for two cases that are distinguishable by the Status column: (1) gap diagnoses (❌ No Test Code, ❌ No Impl, etc.) where "—" indicates the absence that defines the diagnosis; (2) Unknown diagnosis (⚠️ with "—") where the AC has implementation + test code but Phase 6 produced no evidence record.

> **Why L4 (highest-numbered) is "Weak" in isolation**: L4 confirms the *report's accuracy* (metadata about evidence), not the AC's *test automation depth*. L4 alone means "the verifier says the report is correct, but the AC has no automated test coverage." Automated test evidence (L1/L2) is preferred because it is repeatable, objective, and regression-aware. Manual/L4-only evidence is "Weak" because it cannot be re-executed cheaply and may degrade over time. An AC with L1 or L2 evidence is automatically "Covered" regardless of L4 status.
>
> **L4 write mechanism**: L4 is NOT set during Parts 1–3 (matrix construction) — it cannot exist before Part 4 runs. Instead: (1) Parts 1–3 assign L1/L2/L3 to matrix rows based on Phase 6/7 evidence; (2) Part 4 verifies the report; (3) Part 4's outcome is recorded in the Report Status field (`original` or `corrected (N factual errors fixed)`) — this is the L4 evidence. L4 does NOT need a separate row in the Evidence Level column; it is captured through the Report Status field.

Note: Phase 6 Sub-Phase 1.5 (Soak), 1.6 (Contract), and 2 (Cross-Cutting) are system-level checks, not per-AC evidence — they don't map to individual AC rows. Sub-Phase 2 results are excluded for the same reason as 1.5/1.6: they validate system-wide properties (config consistency, CI reproducibility, regression guard) rather than specific AC compliance. Results from these system-level checks are captured in the Phase 6 Release Gate Checklist and reported in the Phase 6 System-Level Checks section of the Part 3 report (see Part 3). A failing system-level check may indicate a broader quality issue that affects multiple ACs even though it does not appear in any single AC's evidence row.

---

## Part 2: Coverage Verification

For each AC in the traceability matrix, diagnose its coverage status. **If any core AC triggers a Phase 6 re-run (Unknown diagnosis), complete the Phase 6 re-run first, then reconstruct the traceability matrix with updated evidence before finalizing Parts 2–3.** The re-run produces new L1/L2 evidence that changes downstream diagnoses. For secondary Unknowns, record as warning and continue Parts 2–3 without pausing; if the user later chooses to resolve in Part 5, re-run Phase 6 and reconstruct Parts 2–3 in a subsequent cycle.

### Diagnosis Table

Evaluate in order — first matching diagnosis wins:

| Diagnosis | Condition | Action |
|-----------|-----------|--------|
| ❓ Bad Requirement | Requirement is infeasible or self-contradictory | rollback to Phase 1 |
| ❌ No Design | Phase 2 has no component mapping for this AC | rollback to Phase 2 |
| ❌ No Test Plan | Phase 3 Requirements Coverage Matrix has no row for this AC | rollback to Phase 3 |
| ❌ No Test Code | Phase 3 has a row but no corresponding test file, OR test file exists but the specific test function named in Phase 3's row is absent from the file (file-level existence alone is insufficient — the named test function must be present), OR test file exists but contains only scaffold placeholders with TODO comments and no meaningful test functions | rollback to Phase 4 |
| ❌ No Impl | Phase 2 has component mapping but component has no code file (a file with ≥1 meaningful function/class/export — scaffold placeholders with only TODO comments do not count) | rollback to Phase 5 (also verify Phase 4 test files exist; if missing, rollback to Phase 4 instead — root cause may be Phase 4, not Phase 5) |
| ⚠️ Weak | implementation exists but only L3 evidence (no L1/L2) | core→BLOCKER (unless inherently non-automatable, document justification); secondary→accepted up to 20% (see Secondary Threshold Check); excess→warning (see Rollback Paths: Weak core → Phase 4, Secondary warning → No rollback). *Core only: If Phase 3 planned only manual testing for this AC (not automated), rollback to Phase 3 first (to plan automated tests), then proceed through Phase 4→5→6→7→8.* **Test-code check** (sub-dispatch within Weak's action — does not re-evaluate the diagnosis order): If automated test code already exists for this AC (Phase 3 Requirements Coverage Matrix lists a test case AND the corresponding test file from Phase 4 exists with the named test function), the root cause is Phase 6 not running/capturing the test — treat as Unknown (full Phase 6 re-run from Sub-Phase 0) rather than Weak. **When reclassifying Weak→Unknown**: update the AC's Status from ⚠️ to ⚠️†, Evidence Level to "—", and record Unknown (not Weak) in the Blockers table. |
| ⚠️ Unknown | implementation + test code exist but no Phase 6 evidence found | Full re-run Phase 6 from Sub-Phase 0 (Phase 6 always re-runs from its Sub-Phase 0); if evidence remains missing after full re-run, escalate to user |
| ✅ Covered | test case exists + Phase 6 pass + implementation exists | PASS |

> **Test-to-AC correctness**: The "Covered" diagnosis verifies that a test exists and passes. It does NOT verify that the passing test correctly validates the AC's requirements. Part 4 (Independent Verification) spot-checks this by reading test code and confirming it exercises the AC's "Given/When/Then" scenario. If Part 4 finds a test that passes but tests the wrong behavior, it records this as a factual error (not a BLOCKER in Part 2).

**Multi-component ACs**: When an AC maps to multiple components (per Phase 2's AC→Component mapping), evaluate each component independently against the Diagnosis Table conditions, then apply the **weakest-link rule**: the AC's overall diagnosis equals the worst diagnosis among its components. Evidence Level uses the lowest evidence level across components (e.g., Component A = L1, Component B = L3 → AC Evidence Level = L3, diagnosis = Weak). List all components' evidence in the Test Case(s) column (comma-separated) so the mixed coverage is visible. A component with no test and another with L1 is diagnosed as the gap diagnosis (e.g., No Test Code), not Covered — the untested component's gap takes precedence over the tested component's pass. **Scope note**: "first matching wins" applies within a single component's diagnosis evaluation; "weakest-link" applies across components of the same AC. These are different scopes, not conflicting rules.

**Missing requirement detection**: Use Phase 2's AC→Component mapping to identify which components serve each AC, then locate those components' code files through the project's source directory structure (Phase 5 output). Do not keyword-grep Phase 5 code for requirement text.

> **Pipeline integrity assumptions**: The diagnosis table assumes Phase 6 gate passed (all tests passing) and Phase 7 gate passed (no unresolved C/H findings). A "test exists + implementation exists + Phase 6 FAIL" scenario indicates a pipeline violation (Phase 6 gate bypassed or Phase 7 fix without re-run). Treat as Unknown diagnosis (full Phase 6 re-run from Sub-Phase 0). Unresolved C/H from Phase 7 should not occur (defensive check in Phase 7 findings table, category 3).

**Unresolved Phase 7 findings** — Phase 7 findings are handled in four categories:

| Category | Action | Risk Assessment |
|----------|--------|-----------------|
| (1) Fixed findings | Reflected as implementation evidence in relevant AC rows (see Phase 7 fixed issues rule above) | N/A |
| (2) Unresolved M/L findings | Listed in Phase 7 Impact section with full risk assessment | Required: finding ID, potential impact on evidence validity, recommended user attention level |
| (3) Unresolved C/H findings | Treated as BLOCKER (defensive check: should not occur after Phase 7 gate). Rollback to Phase 7 → apply rollback guidance → re-run from target through 7→8. Do not complete Parts 3–5; initiate rollback immediately after Part 2 diagnosis. | N/A (BLOCKER) |
| (4) Unresolved P findings | Noted in Phase 7 Impact section for completeness only | Not required (informational) |

> **Why M/L gets risk assessment but P does not**: Pipeline severity ranks P (Petty) above L (Low), but P findings are cosmetic (formatting/wording) with no functional risk. L findings, while lower-ranked, may carry observational insights relevant to the go/no-go decision. The grouping follows risk-relevance (M and L require assessment; P is informational), not severity rank order.

**User attention levels** for unresolved findings below C/H severity (defined here for Phase 8 report presentation; Phase 7 severity classification is the source of truth for finding severity):

| Phase 7 Severity | Phase 8 Attention Level | Meaning |
|-----------------|------------------------|---------|
| M (Medium) | **high** | Review before release decision — may indicate a latent defect or incomplete fix |
| P (Petty) | **medium** | Acknowledge awareness — cosmetic/formatting concern, no functional risk |
| L (Low) | **low** | Informational, no action required — observational insight only |

**Phase 7 findings → AC mapping**: For each Phase 7 finding, identify affected components via file:line references, then use Phase 2's AC→Component mapping to find associated ACs. Findings whose affected components are not in Phase 2's Component Table, or that have no specific component reference, are system-level and noted separately in the Phase 7 Impact section.

**Secondary Threshold Check**: After diagnosing all ACs, count secondary ACs with `Weak` diagnosis. If this count exceeds 20% of ALL secondary ACs in the project, list the excess in the Warnings section of the report. **Selection for accepted vs excess**: among secondary Weak ACs, the first N (where N = floor(20% × total_secondary_ACs)) are accepted (✱), ordered by their AC# number (lowest first); remaining are excess and listed in Warnings. (Edge case: if the project has zero secondary ACs, this check is vacuous — skip it.)

### Core AC Rules

⛔ `Bad Requirement` → BLOCKER (rollback to Phase 1)
⛔ `No Design` / `No Test Plan` / `No Test Code` / `No Impl` → BLOCKER (each per Rollback Paths table below)
⛔ `Weak` (only L3, no L1/L2) → BLOCKER, unless the AC is inherently non-automatable (document justification in report). Rollback to Phase 4 (write automated tests), then re-run Phase 4→5→6→7→8. *Exception: If Phase 3 planned only manual testing for this AC (not automated), rollback to Phase 3 first (to plan automated tests), then proceed through Phase 3→4→5→6→7→8.* **Note**: "Non-automatable" does not exempt Phase 4 test code — write tests that capture state/output for manual review (e.g., screenshot capture, hardware state dump, API response logging). The justification explains why L1/L2 automated assertion is infeasible, not why test code should not exist.
⛔ `Unknown` (implementation + test code exist but no Phase 6 evidence) → BLOCKER (full Phase 6 re-run from Sub-Phase 0; if evidence remains missing after re-run, escalate to user)
✅ `Covered` (L1 or L2) → PASS

### Secondary AC Rules

`No Test Plan` / `No Test Code` / `No Impl` / `No Design` → warning (recorded in report, user decides)
`Bad Requirement` → warning (recorded in report, user decides whether to rollback to Phase 1)
`Unknown` → warning (recorded in report; Phase 6 re-run if user chooses to resolve — secondary Unknown does NOT force a Phase 6 re-run, unlike core Unknown which is a BLOCKER)
`Weak` → max 20% of secondary ACs may have only L3 evidence (advisory threshold: the 20% figure is a guideline for acceptable manual-only coverage; the hard gate is that all excess above 20% must be listed as warnings — the threshold does not block release but ensures visibility); excess → warning
✅ `Covered` → PASS

---

## Part 3: Acceptance Report

### Format

```markdown
# Acceptance Report

**Report Status**: original | corrected (N factual errors fixed)

## Coverage Summary
- Core ACs: X out of Y covered (Z blockers, W non-automatable with justification)
- Secondary ACs: X out of Y covered, W accepted manual-only (Z warnings)

## Traceability Matrix
[full matrix from Part 1]

## Blockers
| Entity | Type | US# | Diagnosis | Root Cause Phase |
|--------|------|-----|-----------|-----------------|
| [Part 2 diagnosis: AC ID (e.g., AC-3)] | AC | [parent US] | [diagnosis from table] | [phase per rollback paths] |
| [Phase 7 unresolved C/H: Finding ID (e.g., F-42)] | Phase 7 | system | Phase 7 C/H | [per Phase 7 rollback guidance] |
| [Phase 6 system-level FAIL: Sub-Phase + check name (e.g., SP2: Config consistency)] | System | system | Phase 6 system-level | Phase 6 |

## Warnings
| AC# | US# | Priority | Diagnosis | Evidence Level | Notes |
|-----|-----|----------|-----------|----------------|-------|
| [secondary AC coverage gaps] |
| [cross-validation gaps: one row per parent AC (if component serves multiple ACs, create one row for each); parent US# in US#, Priority from AC, "Cross-validation gap" in Diagnosis, "—" in Evidence Level, "no test case for component [name]" in Notes] |
| *(summary)* | — | secondary | Weak threshold exceeded (X/Y secondary ACs, 20% max) | — | Z excess ACs listed below |
| [excess Weak secondary ACs] |

## Non-Automatable Core ACs (if any)
| AC# | Component(s) | Evidence Level | Justification Category | Justification Detail |
|-----|--------------|----------------|----------------------|---------------------|
| [ACs with L3 only (no L1/L2)] | [mapped components] | L3 | [one of: (1) visual/appearance verification requiring human judgment; (2) hardware-specific behavior not reproducible in test environment; (3) third-party service with no sandbox/staging API — no other categories accepted] | [specific reason why automated testing is infeasible for this AC] |

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
| [Sub-Phase 1.5 (Soak) / 1.6 (Contract) / 2 (Cross-Cutting) — use Sub-Phase number, not just check name; Release Gate items inherit Sub-Phase from their evidence source] | [check name] | [PASS/FAIL] | [from Phase 6 Release Gate Checklist; system-level FAIL → BLOCKER, rollback to Phase 6 (full re-run from Sub-Phase 0); warnings are informational] |
```

---

## Part 4: Independent Verification

An independent subagent (oracle or dedicated verifier, not involved in Parts 1–3) receives the acceptance report + Phase 1–3 documents (including Phase 2 Component Table for component isolation verification) + Phase 6 test results + Phase 7 audit report + test code files + implementation source code. **Isolation requirement**: the Part 4 agent must NOT receive Parts 1–3 intermediate reasoning, draft matrices, or diagnostic deliberations — only the final artifacts listed above. **Phase 6 re-run dependency**: if Part 2 triggered a Phase 6 re-run (Unknown diagnosis), Part 4 receives the post-re-run Phase 6 results (the same results used to construct the final traceability matrix).

### Verification Checklist

- [ ] Matrix completeness: AC count matches Phase 1 (no ACs silently dropped or added)
- [ ] Test existence: for BLOCKER-status ACs with non-empty Test Case(s), verify ALL listed test cases exist (file + function name). For BLOCKER-status ACs with Test Case(s) = "—", verify the diagnosis correctly identifies the gap (e.g., confirm no test file exists for No Test Code diagnosis). For all non-BLOCKER ACs (Covered + Weak + secondary-priority ACs with warning diagnoses), verify ≥ 20% random sample (minimum 5, or all if fewer than 5), selected uniformly at random using a deterministic seed (e.g., hash of AC IDs modulo sample size) for reproducibility
- [ ] Implementation existence: for a random sample of Covered ACs (≥5 or all if fewer), verify ≥1 source file exists for each mapped component
- [ ] Evidence accuracy: L1 claims trace to Phase 6 Sub-Phase 0 records; L2 to Sub-Phase 1; L3 to Sub-Phase 3 manual exploratory checklist results; L4 to Part 4 Report Status field (original or corrected)
- [ ] Classification correctness: core/secondary labels match Phase 1
- [ ] Rollback target correctness: each BLOCKER's root cause phase matches the actual gap
- [ ] No Test Plan vs No Test Code: diagnosis correctly distinguishes the two
- [ ] Weak diagnosis accuracy: for each AC diagnosed as Weak, verify no automated test code exists that would trigger reclassification to Unknown per the Weak sub-dispatch rule (test-code check). If test code does exist, the AC should be diagnosed as Unknown, not Weak
- [ ] Non-automatable justification: each Core AC claiming L3 only (no L1/L2) has specific, defensible justification (not generic "hard to test"). Valid justifications include: (1) visual/appearance verification requiring human judgment; (2) hardware-specific behavior not reproducible in test environment; (3) third-party service with no sandbox/staging API — no other categories accepted
- [ ] Phase 7 fix reflection: if Phase 7 found and fixed bugs, verify (1) the fixes are reflected as implementation evidence in the relevant AC rows, and (2) Evidence Level was upgraded to L1/L2 if Phase 6 re-run was performed after the fix
- [ ] Phase 7 Impact completeness: all unresolved findings from the Phase 7 audit report (below C/H severity) are listed with their affected ACs. System-level findings (no specific AC mapping) are listed in the System-level findings section, not in the per-AC findings table. For M/L findings, verify risk assessment includes finding ID, potential impact, and recommended attention level. For P findings, verify they are listed (informational only, no risk assessment required)
- [ ] Secondary threshold accuracy: count of Weak secondary ACs matches report; >20% threshold correctly applied; excess listed in Warnings section
- [ ] Cross-validation gap identification: for each cross-validation gap flagged in Part 1 (no test case for a component), verify the gap is listed in the Warnings table with correct parent AC# and component name
- [ ] Phase 6 System-Level Checks accuracy: results in the Phase 6 System-Level Checks section match Phase 6 Release Gate Checklist (PASS/FAIL status, Sub-Phase, check name). System-level FAIL entries also appear in the Blockers table with correct format.

### Gate

⛔ Independent verification fails → correct the report (factual errors only). No rollback. Re-verify corrected report. **Re-verification scope**: if corrections affect ≤3 AC rows, re-verify only the affected rows plus any dependent Blockers/Warnings entries. If corrections affect >3 rows or change the Coverage Summary counts, re-verify the full report. Maximum 3 correction-verification cycles; if factual errors persist after 3 cycles, escalate to user for manual review.

---

## Part 5: User Go/No-Go

The main agent (who executed Parts 1–3) receives the verified report from Part 4 and presents it to the user.

Submit to user: acceptance report (Part 3, verified by Part 4) + Phase 6 Release Gate Checklist + Phase 7 audit report.

| Condition | Action |
|-----------|--------|
| Zero BLOCKERs | User Go/No-Go decision (include Phase 7 Impact section with unresolved M/L findings and attention levels; recommend user reviews all "high" attention findings before deciding Go) |
| Any BLOCKER | Rollback per diagnosis table → re-run pipeline from rollback target |
| User Go | Release: tag + push + release |
| User No-Go | Shelve or rollback to user-specified phase |

---

## Rollback Paths

| Diagnosis | Root Cause | Rollback To | Re-run Scope |
|-----------|-----------|-------------|-------------|
| Bad Requirement | Phase 1 | Phase 1 | Full pipeline |
| No Design | Phase 2 | Phase 2 | Phase 2→3→4→5→6→7→8 |
| No Test Plan | Phase 3 | Phase 3 | Phase 3→4→5→6→7→8 |
| No Test Code | Phase 4 | Phase 4 | Phase 4→5→6→7→8 |
| No Impl | Phase 5 | Phase 5 | Phase 5→6→7→8 (also verify Phase 4 test files exist; if missing, rollback to Phase 4 instead) |
| Weak (core) | Phase 4 (or Phase 3) | Phase 4 (or Phase 3) | Phase 4→5→6→7→8 (unless inherently non-automatable — documented justification required; if Phase 3 planned only manual testing → Phase 3→4→5→6→7→8) |
| Unknown | Phase 6 | Phase 6 | Phase 6→7→8 (full re-run from Sub-Phase 0; if evidence remains missing after re-run, escalate to user) |
| Secondary warning | — | No rollback (default) | Recorded in report, user decides. If user chooses to rollback: Bad Requirement → Phase 1, other secondary gaps → Phase per diagnosis table |
| Secondary: Bad Requirement | Phase 1 | Phase 1 (user choice) | Full pipeline — same path as core, but user-initiated |
| Secondary: No Design | Phase 2 | Phase 2 (user choice) | Phase 2→3→4→5→6→7→8 — same path as core, but user-initiated |
| Secondary: No Test Plan | Phase 3 | Phase 3 (user choice) | Phase 3→4→5→6→7→8 — same path as core, but user-initiated |
| Secondary: No Test Code | Phase 4 | Phase 4 (user choice) | Phase 4→5→6→7→8 — same path as core, but user-initiated |
| Secondary: No Impl | Phase 5 | Phase 5 (user choice) | Phase 5→6→7→8 — same path as core, but user-initiated |
| Secondary: Weak | Phase 4 | No rollback (default) | Accepted up to 20% of secondary ACs; excess → warning only |
| Secondary: Unknown | Phase 6 | No rollback (default) | Warning — recorded in report. Phase 6 re-run only if user chooses (not forced like core Unknown) |
| Phase 7 unresolved C/H | Phase 7 (via Phase 6 root cause) | Per Phase 7 rollback guidance (Phase 7 delegates to Phase 6 root cause categories: test gap, code bug, design flaw, requirement misunderstanding, config/environment) | Rollback to Phase 7 → apply Phase 7's root cause layer → target phase → re-run from target through Phase 7→8. Note: this is a two-level indirection — Phase 8 detects the issue, Phase 7 determines the root cause layer, Phase 6 (or earlier) is the actual rollback target. |
| Phase 6 system-level failure | Phase 6 (system-level check) | Phase 6 | Phase 6→7→8 (full re-run from Sub-Phase 0; system-level failures bypass AC-level diagnosis — they indicate a systemic quality issue requiring full Phase 6 re-execution). "System-level failure" = a Phase 6 Sub-Phase 1.5/1.6/2 check that FAILs (these validate system-wide properties like config consistency, CI reproducibility, and regression guards — see `phase-6-pre-release-testing.md` for definitions). |

**Rules:** (1) Rollback preserves upstream phase artifacts. (2) All quality mechanisms from rollback point onward must re-run (Ralph loops for Phases 1–5, sub-phase gates for Phase 6, audit for Phase 7, acceptance for Phase 8). (3) Phase 6 always full re-run from its Sub-Phase 0. (4) Phase 7 full re-run after Phase 6 passes. (5) Phase 8 re-runs after Phase 7 passes (full traceability reconstruction). (6) **Multiple BLOCKERs** targeting different phases → rollback to the earliest (lowest-numbered) target phase; this subsumes all later-phase rollbacks. Exception: Phase 7 unresolved C/H BLOCKERs follow Phase 7's own rollback guidance (which may target a specific Phase 6 root cause layer), not the generic earliest-phase rule — Phase 7's domain-specific guidance takes precedence. **When Phase 8 AC BLOCKERs and Phase 7 C/H BLOCKERs coexist**: collect all rollback targets; for Phase 8 AC BLOCKERs, apply earliest-phase rule; for Phase 7 C/H BLOCKERs, follow Phase 7 guidance; rollback to the earliest target across both sets.

---

## Next Step

User Go → release per project's deployment workflow.
