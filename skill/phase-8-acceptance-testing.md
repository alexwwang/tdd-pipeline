---
name: acceptance-testing
description: Functional acceptance with requirements traceability. Loaded after Phase 7.
---

# Acceptance Testing — Phase 8: Functional Verification

**Core Principle**: Phase 8 = completeness, not correctness. Phase 6/7 verified correctness; Phase 8 verifies completeness.

**Scope Boundary**: Phase 8 diagnoses coverage gaps and reports findings to the user for decision. It does NOT initiate upstream phase re-runs, prescribe rollback targets, or define quality standards for upstream deliverables.

**Edge cases at load**:
- **Zero ACs**: Skip Parts 1–3, proceed to Part 5 with empty report. **Phase 7 unresolved C/H and Phase 6 system-level FAILs are BLOCKERs regardless of AC count** — check even with zero ACs.
- **Zero core ACs (only secondary)**: Run Parts 1–4 normally; Blockers table has no core-AC rows (Phase 7 C/H and Phase 6 system-level BLOCKERs still appear independently).
- **Normal case**: Proceed with Parts 1–5.

**Execution**: Parts 1–3 by main agent or dedicated subagent. Part 4 by different agent. **Load dependencies**: Phase 8 references `phase-7-system-quality-audit.md`. **Split projects**: merge per-module Phase 3 matrices into unified traceability matrix; use Phase 2's integrated Component Table as cross-module reference. **AC renumbering**: if duplicate AC identifiers across modules, renumber sequentially in Phase 2 component map order (AC-3a→AC-3, AC-3b→AC-4, etc.). **Merge conflicts**: use module whose Phase 2 Component Table owns the AC's primary component; if ambiguous, flag for user. **Non-conflicting data**: union all test cases and evidence.

---

## Part 1: Traceability Matrix Construction

### Data Sources

| Source | Extract |
|--------|---------|
| Phase 1 | All User Stories + Acceptance Criteria (core/secondary) |
| Phase 2 | AC→Component mapping (from "Serves Phase 1 ACs" column; bidirectional for cross-validation) |
| Phase 3 | Requirements Coverage Matrix (AC→Test Case) + Design Coverage Matrix (Component→Test Case) |
| Phase 6 | Sub-Phase pass records (test case → pass/fail) + Release Gate Checklist + re-run metadata (if re-run after Phase 7 fix, must include `re-run: true` or timestamp — distinguishes "never re-run" from "re-run produced no evidence") |
| Phase 7 | Audit report (fixed/rejected items as evidence) + unresolved findings list (defensive C/H check) |

### Merge Rules

From Phase 3's Requirements Coverage Matrix: `Test File` + `Test Name` → `Test Case(s)` column: `test_file::test_name`. Normalize to filenames only (strip directory paths; strip only final extension — keep `.test`/`.spec` in base name). **"—"** = no named test case. Mixed coverage: `test_case (evidence_level), — (evidence_level)`.

Phase 3 Design Coverage cross-validation: for each AC row, cross-check that components have ≥1 test case. Flag gaps → Warnings in Part 3 ("Cross-validation gap: no test case for component").

Phase 6 mapping: only **pass** results = L1. FAIL/SKIP = no L1 (SKIP from scaffold tests with `skip` markers expected for non-automatable ACs). Sub-Phase 1 integration/E2E = L2. For multi-AC integration tests: assign L2 to all validated ACs; partial coverage = note in report, no full L2 unless all components covered. If Phase 6 results are not in structured format, report affected ACs as Unknown.

Phase 7 fixed issues: if Phase 6 re-run results available AND show L1/L2, upgrade Evidence Level. If no re-run, retain original level + note fix in Phase 7 Impact.

### Matrix Format

```
| AC# | US# | Priority | AC Description | Component(s) | Test Case(s) | Evidence Level | Status | Notes |
|-----|-----|----------|---------------|--------------|--------------|----------------|--------|-------|
| AC-6 | US-4 | core | Given X When Y Then Z | OrderAPI, OrderDB | test_order::should_create (L1), — (L3) | L3 | ⚠️ | Multi-component weakest-link |
```

\* Secondary ❌ = warning (not BLOCKER). † Unknown (impl + test exist but no Phase 6 evidence). ‡ Multi-component weakest-link: min(L1, L3) = L3.

**Status symbols**: ✅ Covered | ❌ gap (core=BLOCKER, secondary=warning) | ⚠️ Weak (L3 evidence) or Unknown (⚠️†) | ✱ accepted secondary Weak (within 20% threshold)

### Evidence Levels

**Ranking**: L1 > L2 > L3 > —. **Highest-within** component, **weakest-across** components.

| Level | Source |
|-------|--------|
| L1 | Phase 6 Sub-Phase 0 (unit test pass) |
| L2 | Phase 6 Sub-Phase 1 (integration/E2E pass) |
| L3 | Phase 6 Sub-Phase 3 (manual exploratory checklist) |
| L4 | Part 4 verification (Report Status field — metadata about report accuracy, NOT per-AC test evidence) |

**"—"** in Evidence Level: gap diagnosis (no evidence) or Unknown (⚠️†, impl+test exist but no Phase 6 record) — distinguish by Status column.

Note: Phase 6 Sub-Phase 1.5/1.6/2 are system-level, NOT per-AC evidence — reported in Phase 6 System-Level Checks section.

---

## Part 2: Coverage Verification

For each AC, diagnose coverage status. **Unknown (core)** = BLOCKER — user decides whether to trigger Phase 6 re-run. Do NOT initiate re-runs.

### Diagnosis Table

Evaluate in order — first matching diagnosis wins:

| Diagnosis | Condition | Verdict |
|-----------|-----------|---------|
| ❓ Bad Requirement | Requirement infeasible or self-contradictory | core→BLOCKER; secondary→warning |
| ❌ No Design | Phase 2 has no component mapping | core→BLOCKER; secondary→warning |
| ❌ No Test Plan | Phase 3 Requirements Coverage Matrix has no row | core→BLOCKER; secondary→warning |
| ❌ No Test Code | Phase 3 has row but test file absent from test directories, OR test function absent from file. Multi-component: fires if ANY listed file missing | core→BLOCKER; secondary→warning |
| ❌ No Impl | Phase 2 has component mapping but no source code file outside test directories | core→BLOCKER; secondary→warning |
| ⚠️ Weak | impl exists but no L1/L2 (Evidence Level = L3 or "—") | core→BLOCKER (unless non-automatable); secondary→accepted up to 20%; excess→warning |
| ⚠️ Unknown | (via Weak→Unknown reclassification only) | core→BLOCKER (user decides); secondary→warning |
| ✅ Covered | test exists + L1/L2 evidence + impl exists | PASS |

> **Escalated Unknown**: Unknown becomes "Escalated" when Phase 6 was re-run and evidence still missing. Three options: accept risk, manual investigation, or shelve. Recorded in Accepted Risks.

> **Test-to-AC correctness**: Covered = test exists and passes, NOT that test correctly validates AC. Part 4 spot-checks this. If test tests wrong behavior: re-diagnose from Bad Requirement downward; if test maps to different AC, remove mapping and re-diagnose.

### Weak→Unknown Reclassification (sub-dispatch)

After Weak matches via first-matching-wins, check reclassification:

**Condition**: Evidence Level = "—" (zero Phase 6 evidence) AND test code exists (Phase 3 lists test + Phase 4 file exists with named function).

**Action**: Reclassify to Unknown. Status ⚠️ → ⚠️†. Record Unknown in Blockers.

**Does NOT fire when**: Evidence Level = L3 (Weak is genuine).

**Core-only exception**: If Phase 3 planned only manual testing, root cause is Phase 3.

### Multi-component ACs

**Weakest-link rule**: AC overall diagnosis = worst among components. **Severity order** (worst→best): Bad Requirement > No Design > No Test Plan > No Test Code > No Impl > Weak > Unknown > Covered.

Evidence Level = lowest across components. Format: `test_case (evidence_level)` per component, comma-separated.

> **Pipeline integrity**: Assumes Phase 6/7 gates passed. "test exists + impl exists + Phase 6 FAIL" = pipeline violation → report as Unknown with note.

### Phase 7 Findings

| Category | Action | Risk Assessment |
|----------|--------|-----------------|
| (1) Fixed | Reflected as implementation evidence in AC rows | N/A |
| (2) Unresolved M/L | Phase 7 Impact section with full risk assessment | Required: finding ID, potential impact, attention level |
| (3) Unresolved C/H | BLOCKER. Do not complete Parts 3–5; report to user with abbreviated report | N/A (BLOCKER) |
| (4) Unresolved P | Phase 7 Impact section, informational | Not required |
| (5) Rejected | Phase 7 Impact section, informational | Not required |

**Attention levels** (for report presentation):

| Phase 7 Severity | Attention Level | Meaning |
|-----------------|----------------|---------|
| M (Medium) | **high** | Review before release — may indicate latent defect |
| P (Petty) | **medium** | Acknowledge — cosmetic, no functional risk |
| L (Low) | **low** | Informational only |

### Secondary Threshold Check

After diagnosing all ACs, count secondary Weak. N = floor(20% × total_secondary). First N Weak secondaries (ordered by AC# numeric suffix, lowest first) = accepted (✱). Excess → Warnings.

### Core AC Rules

⛔ `Bad Requirement` / `No Design` / `No Test Plan` / `No Test Code` / `No Impl` → BLOCKER
⛔ `Weak` (L3 evidence after sub-dispatch) → BLOCKER, unless inherently non-automatable. **Expected test code form**: (1) visual → scaffold with `skip` + manual procedure comments; (2) hardware → scaffold with mock interfaces; (3) third-party → scaffold with recorded fixtures.
⛔ `Unknown` → BLOCKER (report to user)
✅ `Covered` → PASS

### Secondary AC Rules

`No Test Plan` / `No Test Code` / `No Impl` / `No Design` / `Bad Requirement` / `Unknown` → warning
`Weak` → max 20% accepted; excess → warning
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

> **"Release-ready"**: ✅ Covered, ✱ accepted secondary Weak, or ⚠️ non-automatable core with justification. ❌ and ⚠️† are not release-ready.

## Traceability Matrix
[full matrix from Part 1]

## Blockers
| Entity | Type | US# | Diagnosis |
|--------|------|-----|-----------|
| [AC ID] | AC | [parent US] | [diagnosis] |
| [Phase 7 C/H: Finding ID] | Phase 7 | system | Phase 7 C/H |
| [Phase 6 system FAIL: Sub-Phase + check] | System | system | Phase 6 system-level |

## Warnings
| AC# | US# | Priority | Diagnosis | Evidence Level | Notes |
|-----|-----|----------|-----------|----------------|-------|
| [secondary gaps] |
| [cross-validation gaps: one row per parent AC] |
| — | — | secondary | Weak threshold exceeded (X/Y, 20% max) | — | Z excess below |
| [excess Weak secondary ACs] |

## Non-Automatable Core ACs (if any)
| AC# | Component(s) | Evidence Level | Justification Category | Justification Detail |
|-----|--------------|----------------|----------------------|---------------------|
| [L3-only ACs] | [components] | L3 | [visual/hardware/third-party/other] | [why automated testing infeasible] |

## Phase 7 Impact (if any)
**Fixed findings**: [Finding ID → AC# → Evidence Level upgrade or "no upgrade (re-run pending)"]

**Unresolved findings**:
| Finding ID | Severity | Affected AC# | Potential Impact | User Attention Level |
|------------|----------|--------------|-----------------|---------------------|
| [M/L — full risk assessment] |
| [P — informational] |

**System-level findings** (no AC mapping):
[Finding ID → Severity → Description → Attention Level]

## Phase 6 System-Level Checks
| Sub-Phase | Check | Result | Notes |
|-----------|-------|--------|-------|
| [SP 1.5/1.6/2 + check name] | [check] | [PASS/FAIL] | [from Release Gate Checklist] |

## Accepted Risks (if any)
| AC# | US# | Original Diagnosis | Escalation Context | User Decision | Risk Acknowledgment |
|-----|-----|--------------------|--------------------|---------------|-------------------|
| [escalated Unknowns accepted via Part 5 option (a)] |
```

---

## Part 4: Independent Verification

Independent subagent (not involved in Parts 1–3) receives: acceptance report + Phase 1–3 documents + Phase 6 results + Phase 7 report + test code + implementation source. **Isolation**: Part 4 agent must NOT receive Parts 1–3 intermediate reasoning or drafts.

### Verification Checklist

- [ ] Matrix completeness: AC count matches Phase 1
- [ ] Test existence — BLOCKERs: for gap diagnoses, confirm test does NOT exist; for non-gap, confirm ALL tests exist
- [ ] Test existence — non-BLOCKERs: ≥20% random sample, minimum 5, seeded PRNG for reproducibility
- [ ] Implementation existence: ≥5 random Covered ACs, verify ≥1 source file per component
- [ ] Evidence accuracy: L1→Sub-Phase 0, L2→Sub-Phase 1, L3→Sub-Phase 3, L4→Report Status
- [ ] Classification correctness: core/secondary matches Phase 1
- [ ] No Test Plan vs No Test Code: correctly distinguished
- [ ] Weak accuracy: each Weak has L3 (not "—"); if "—" + test exists → should be Unknown (⚠️†)
- [ ] Non-automatable justification: specific and defensible (not generic "hard to test")
- [ ] Phase 7 fix reflection: fixes reflected + Evidence Level upgraded only if re-run performed
- [ ] Phase 7 Impact completeness: all unresolved findings listed; M/L with risk assessment; P informational
- [ ] Secondary threshold accuracy: count matches; >20% threshold correct; excess in Warnings
- [ ] Cross-validation gaps: flagged in Part 1 listed in Warnings with correct parent AC#
- [ ] Phase 6 System-Level Checks: match Release Gate Checklist; FAIL entries also in Blockers

### Gate

⛔ Verification fails → correct report (factual errors only). Re-verify. If ≤3 rows affected: re-verify affected rows + dependent entries only (Blockers/Warnings referencing same AC#; if secondary status changed, re-verify threshold summary). If >3 rows or Coverage Summary changed: full re-verify. Max 3 cycles; then escalate to user.

---

## Part 5: User Go/No-Go

Main agent receives verified report from Part 4, presents to user.

**Step 1: Evaluate BLOCKER state**

| Condition | Action |
|-----------|--------|
| Zero BLOCKERs | Proceed to Step 2 |
| Any BLOCKER (standard) | Present to user for decision |
| Escalated Unknown | Present to user: (a) accept risk → Go/No-Go, (b) manual investigation, (c) shelve |

**Post-verification update for option (a)**: remove from Blockers, add to Accepted Risks with escalation context, update Coverage Summary blocker count (Z−1), update AC Status from ⚠️† to Accepted Risks reference. Report Status unchanged (Part 4 domain).

**Step 2: User Go/No-Go** (zero BLOCKERs only)

| Decision | Phase 8 Reports |
|----------|----------------|
| User Go | Approve release → project's deployment workflow |
| User No-Go | Decline → user specifies next action |
