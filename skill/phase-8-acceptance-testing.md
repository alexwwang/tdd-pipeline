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

---

## Part 1: Traceability Matrix Construction

### Data Sources

| Source | Extract |
|--------|---------|
| Phase 1 | All User Stories + Acceptance Criteria (core/secondary) |
| Phase 2 | Component → code file mapping (AC→Component path) |
| Phase 3 | Requirements Coverage Matrix (AC→Test Case mapping) + Design Coverage Matrix (Component→Test Case) |
| Phase 6 | Sub-phase pass records (test case name → pass/fail) |
| Phase 7 | Bug fix records (fixed issues become new implementation evidence) |

### Merge Rules

From Phase 3's Requirements Coverage Matrix, extract `Test File` and `Test Name` — merge into single `Test Case(s)` column: `test_file::test_name`.

Phase 3's Design Coverage Matrix provides component→test cross-validation: for each AC row, use Phase 2's AC→Component mapping to find components, then cross-check Phase 3 Design Coverage Matrix: the component should have ≥1 test case listed. If not, flag as potential coverage gap.

Phase 6 Sub-Phase 0 output (pytest/vitest) maps test case names to pass/fail status.

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
| L3 | Phase 6 Sub-Phase 3 (manual exploratory record) |
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
| ❌ No Impl | Phase 2 has component mapping but component has no code file | rollback to Phase 5 |
| ❌ No Test Plan | Phase 3 Requirements Coverage Matrix has no row for this AC | rollback to Phase 3 |
| ❌ No Test Code | Phase 3 has a row but no corresponding test file | rollback to Phase 4 |
| ⚠️ Weak | implementation exists but only L3/L4 evidence | core→BLOCKER; secondary→warning |
| ✅ Covered | test case exists + Phase 6 pass + implementation exists | PASS |

**Missing requirement detection**: Use Phase 2's AC→Component mapping to locate code files. Do not keyword-grep Phase 5 code.

### Core AC Rules

⛔ `No Test Plan` / `No Test Code` / `No Impl` / `No Design` → BLOCKER
⛔ `Weak` (only L3/L4) → BLOCKER, unless the AC is inherently non-automatable (document justification in report)
✅ `Covered` (L1 or L2) → PASS

### Secondary AC Rules

`No Test Plan` / `No Test Code` / `No Impl` → warning (recorded in report, user decides)
`Weak` → max 20% of secondary ACs may have only L4 evidence (heuristic: allows reasonable manual-only coverage for UI/UX ACs while preventing systematic test avoidance); excess → warning
✅ `Covered` → PASS

---

## Part 3: Acceptance Report

### Format

```markdown
# Acceptance Report

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
```

---

## Part 4: Independent Verification

An independent subagent (oracle or dedicated verifier, not involved in Parts 1–3) receives the acceptance report + Phase 1–3 documents + source code.

### Verification Checklist

- [ ] Matrix completeness: AC count matches Phase 1 (no ACs silently dropped or added)
- [ ] Test existence: for BLOCKER-status ACs, verify ALL test cases exist (file + function name). For remaining ACs, verify ≥ 20% random sample (minimum 5)
- [ ] Evidence accuracy: L1 claims trace to Phase 6 Sub-Phase 0 records; L2 to Sub-Phase 1
- [ ] Classification correctness: core/secondary labels match Phase 1
- [ ] Rollback target correctness: each BLOCKER's root cause phase matches the actual gap
- [ ] No Test Plan vs No Test Code: diagnosis correctly distinguishes the two
- [ ] Non-automatable justification: each Core AC claiming L3/L4 only has specific, defensible justification (not generic "hard to test")

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
| No Test Plan | Phase 3 | Phase 3 | Phase 3→4→5→6→7→8 |
| No Test Code | Phase 4 | Phase 4 | Phase 4→5→6→7→8 |
| No Impl | Phase 5 | Phase 5 | Phase 5→6→7→8 |
| No Design | Phase 2 | Phase 2 | Phase 2→3→4→5→6→7→8 |
| Bad Requirement | Phase 1 | Phase 1 | Full pipeline |
| Secondary gap | — | No rollback | Recorded in report, user decides |

**Rules:** (1) Rollback preserves upstream phase artifacts. (2) All Ralph loops from rollback point onward must re-run. (3) Phase 6 always full rerun from Phase 0. (4) Phase 7 full rerun after Phase 6 passes.

---

## Next Step

User Go → release per project's deployment workflow.
