---
name: acceptance-testing
description: >
  Functional acceptance testing with requirements traceability. Loaded after
  Phase 7 completes. Traces Phase 1 requirements through Phase 2 components,
  Phase 3-4 test cases, to Phase 5 implementation, with Phase 6-7 evidence.
  AC-level verification with evidence hierarchy, independent check, and
  release decision.
---

# Acceptance Testing — Phase 8: Functional Verification

**Core Principle**: Build the right thing, not just build things right. Phase 6/7 verified correctness. Phase 8 verifies completeness.

**When loaded**: After Phase 7 verification audit passes.

**Execution**: Parts 1–3 (matrix construction, coverage verification, report) are executed by the main agent or a dedicated acceptance subagent. Part 4 must be performed by a different agent to ensure independence.

---

## Part 1: Traceability Matrix Construction

### Data Sources

| Source | Extract |
|--------|---------|
| Phase 1 | All User Stories + Acceptance Criteria (core/secondary) |
| Phase 2 | Component → AC mapping (Serves Phase 1 ACs column from Component Table) |
| Phase 3 | Requirements Coverage Matrix (AC→Test Case mapping) + Design Coverage Matrix (Component→Test Case) |
| Phase 6 | Sub-phase pass records (test case name → pass/fail) |
| Phase 7 | Bug fix records (fixed issues become new implementation evidence) |

Note: Phase 4 (Test Code) and Phase 5 (Business Code) outputs are not listed separately — they are verified indirectly through Phase 6 test results (which run Phase 4 tests) and source code (which contains Phase 5 implementation).

### Merge Rules

From Phase 3's Requirements Coverage Matrix, extract `Test File` and `Test Name` — merge into single `Test Case(s)` column: `test_file::test_name`.

Phase 3's Design Coverage Matrix provides component→test cross-validation: for each AC row, use Phase 2's AC→Component mapping to find components, then cross-check Phase 3 Design Coverage Matrix: the component should have ≥1 test case listed. If not, flag as potential coverage gap.

Phase 6 Sub-Phase 0 output (pytest/vitest) maps test case names to pass/fail status (L1 evidence). If Phase 6 results are not in structured format, re-run with appropriate output flags (`--tb=no -q` for pytest, `--reporter=verbose` for vitest) to extract per-test-case pass/fail. Sub-Phase 1 integration/E2E results provide L2 evidence — extract similarly.

Phase 7 fixed issues: if Phase 7 found and fixed bugs, the fixes are new implementation evidence — add to relevant AC rows.

### Matrix Format

```
| AC# | US# | Priority | AC Description | Component(s) | Test Case(s) | Evidence Level | Status |
|-----|-----|----------|---------------|--------------|--------------|----------------|--------|
| AC-1 | US-1 | core | Given X When Y Then Z | EmailValidator | test_email::should_accept, test_email::should_reject | L1 | ✅ |
| AC-2 | US-1 | core | Given X When Y Then Z | UserService | — | — | ❌ |
```

### Evidence Levels

| Level | Source |
|-------|--------|
| L1 | Phase 6 Sub-Phase 0 (unit test pass) |
| L2 | Phase 6 Sub-Phase 1 (integration/E2E pass) |
| L3 | Phase 6 Sub-Phase 3 (manual exploratory checklist results) |
| L4 | Phase 8 dedicated human confirmation |

Note: Phase 6 Sub-Phase 1.5 (Soak), 1.6 (Contract), and 2 (Cross-Cutting) are system-level checks, not per-AC evidence — they don't map to individual AC rows.

---

## Part 2: Coverage Verification

For each AC in the traceability matrix, diagnose its coverage status:

### Diagnosis Table

Evaluate in order — first matching diagnosis wins:

| Diagnosis | Condition | Action |
|-----------|-----------|--------|
| ❓ Bad Requirement | Requirement is infeasible or self-contradictory | rollback to Phase 1 |
| ❌ No Design | Phase 2 has no component mapping for this AC | rollback to Phase 2 |
| ❌ No Test Plan | Phase 3 Requirements Coverage Matrix has no row for this AC | rollback to Phase 3 |
| ❌ No Test Code | Phase 3 has a row but no corresponding test file | rollback to Phase 4 |
| ❌ No Impl | Phase 2 has component mapping but component has no code file | rollback to Phase 5 |
| ⚠️ Weak | implementation exists but only L3/L4 evidence | core→BLOCKER (unless non-automatable, document justification); secondary→warning (see Rollback Paths for split) |
| ✅ Covered | test case exists + Phase 6 pass + implementation exists | PASS |

**Missing requirement detection**: Use Phase 2's AC→Component mapping to identify which components serve each AC, then locate those components' code files through the project's source directory structure (Phase 5 output). Do not keyword-grep Phase 5 code for requirement text.

**Unresolved Phase 7 findings**: ACs associated with unresolved Phase 7 M/L findings pass their diagnosis but must be listed in the Phase 7 Impact section with a risk assessment (finding ID, potential impact on evidence validity, recommended user attention level). These do NOT trigger BLOCKER status — they are advisory flags for the user's go/no-go decision.

**Phase 7 findings → AC mapping**: For each Phase 7 finding, identify affected components via file:line references, then use Phase 2's AC→Component mapping to find associated ACs. Findings with no specific component reference are system-level and noted separately.

### Core AC Rules

⛔ `Bad Requirement` → BLOCKER (rollback to Phase 1)
⛔ `No Test Plan` / `No Test Code` / `No Impl` / `No Design` → BLOCKER (each per Rollback Paths table below)
⛔ `Weak` (only L3/L4) → BLOCKER, unless the AC is inherently non-automatable (document justification in report). Rollback to Phase 4 (write automated tests), then re-run Phase 4→5→6→7→8.
✅ `Covered` (L1 or L2) → PASS

### Secondary AC Rules

`No Test Plan` / `No Test Code` / `No Impl` / `No Design` → warning (recorded in report, user decides)
`Bad Requirement` → warning (recorded in report, user decides whether to rollback to Phase 1)
`Weak` → max 20% of secondary ACs may have only L3/L4 evidence (heuristic: allows reasonable manual-only coverage for UI/UX ACs while preventing systematic test avoidance); excess → warning
✅ `Covered` → PASS

---

## Part 3: Acceptance Report

### Format

```markdown
# Acceptance Report

**Report Status**: original | corrected (N factual errors fixed)

## Coverage Summary
- Core ACs: X/X covered (Y blockers)
- Secondary ACs: X/X covered (Y warnings)

## Traceability Matrix
[full matrix from Part 1]

## Blockers
| AC# | US# | Priority | Diagnosis | Root Cause Phase |
|-----|-----|----------|-----------|-----------------|

## Warnings
[secondary AC coverage gaps]

## Non-Automatable Core ACs (if any)
[ACs with L3/L4 only, with justification]

## Phase 7 Impact (if any)
[ACs whose evidence changed due to Phase 7 fixes — summarize which ACs were affected and how. Also list ACs affected by unresolved Phase 7 M/L findings — note the finding and its potential impact on evidence validity.]
```

---

## Part 4: Independent Verification

An independent subagent (oracle or dedicated verifier, not involved in Parts 1–3) receives the acceptance report + Phase 1–3 documents + Phase 6 test results + Phase 7 audit report + source code.

### Verification Checklist

- [ ] Matrix completeness: AC count matches Phase 1 (no ACs silently dropped or added)
- [ ] Test existence: for BLOCKER-status ACs, verify ALL test cases exist (file + function name). For remaining ACs, verify ≥ 20% random sample (minimum 5, or all if fewer than 5)
- [ ] Evidence accuracy: L1 claims trace to Phase 6 Sub-Phase 0 records; L2 to Sub-Phase 1; L3 to Sub-Phase 3 manual records; L4 to documented human confirmation
- [ ] Classification correctness: core/secondary labels match Phase 1
- [ ] Rollback target correctness: each BLOCKER's root cause phase matches the actual gap
- [ ] No Test Plan vs No Test Code: diagnosis correctly distinguishes the two
- [ ] Non-automatable justification: each Core AC claiming L3/L4 only has specific, defensible justification (not generic "hard to test")
- [ ] Phase 7 fix reflection: if Phase 7 found and fixed bugs, verify the fixes are reflected as implementation evidence in the relevant AC rows

### Gate

⛔ Independent verification fails → correct the report (factual errors only). No rollback. Re-verify corrected report.

---

## Part 5: User Go/No-Go

Submit to user: acceptance report (Part 3, verified by Part 4) + Phase 6 Release Gate Checklist + Phase 7 audit report.

| Condition | Action |
|-----------|--------|
| Zero BLOCKERs | User Go/No-Go decision |
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
| No Impl | Phase 5 | Phase 5 | Phase 5→6→7→8 |
| Weak (core) | Phase 4 | Phase 4 | Phase 4→5→6→7→8 |
| Secondary warning | — | No rollback | Recorded in report, user decides |

**Rules:** (1) Rollback preserves upstream phase artifacts. (2) All Ralph loops from rollback point onward must re-run. (3) Phase 6 always full re-run from its Sub-Phase 0. (4) Phase 7 full re-run after Phase 6 passes. (5) Phase 8 re-runs after Phase 7 passes (full traceability reconstruction). (6) **Multiple BLOCKERs** targeting different phases → rollback to the earliest (lowest-numbered) target phase; this subsumes all later-phase rollbacks.

---

## Next Step

User Go → release per project's deployment workflow.
