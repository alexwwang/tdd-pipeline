# Ralph Loop Review Protocol

This protocol governs **all reviews** in the TDD Pipeline. It is invoked after every phase (1–5).

## When to Invoke

- **After Phases 1–3 (design phases)**: Launch Ralph loop **design review**
- **After Phases 4–5 (code phases)**: Launch Ralph loop **code review**

## Reviewer Selection

Spawn an **independent subagent** (oracle or dedicated reviewer) that was **not involved** in creating the deliverable. The reviewer must have no prior context of the phase work.

## Severity Classification

Every issue found by the reviewer MUST be classified by severity:

| Severity | Name | Definition | Action |
|----------|------|------------|--------|
| **C** | Critical | Fundamental flaw; deliverable is wrong, dangerous, or useless | Must fix before proceeding |
| **H** | High | Significant gap or serious risk; weakens the deliverable substantially | Must fix before proceeding |
| **M** | Major | Important issue that contradicts requirements or best practices | Must fix before proceeding |
| **L** | Low | Minor improvement, style issue, or optimization | Optional; may carry forward |
| **I** | Info | Observation, question, or suggestion with no defect | No action required |

## Review Process (Per Round)

1. **Present**: Feed the deliverable and relevant prior phase outputs to the reviewer subagent
2. **Review**: The reviewer examines the deliverable against the phase's gate criteria (see individual `phase-N-*.md` files)
3. **Report**: The reviewer outputs a numbered list of issues with severity labels, e.g.:
   - `[H-1]` Missing boundary condition handling in AC-3
   - `[M-2]` Test plan omits integration test for payment webhook
   - `[L-3]` Could use more descriptive variable names in stubs
4. **Tally**: Count issues by severity: `C=0, H=1, M=2, L=3, I=1`
5. **Fix**: Address all C, H, and M issues. L and I issues are optional
6. **Log**: Record the round number, issue tally, and fixes applied

## Rounds & Early Stop Rule

- **Minimum rounds**: 5 rounds (no exceptions)
- **Early stop condition**: ONLY when **2 CONSECUTIVE rounds** report **zero C/H/M/L issues** (i.e., only I or nothing)
  - Must satisfy: `round N = 0 non-I issues` AND `round N+1 = 0 non-I issues`
  - The zero-issue rounds must be **back-to-back with no gap**
- If early stop never occurs, continue until the 5-round minimum is met, then proceed if the final round has zero C/H/M issues

### ⛔ AVOID — Common Early Stop Mistakes (READ CAREFULLY)

These are the most frequent errors LLMs make. **DO NOT do any of these:**

| ❌ WRONG | Why It's Wrong | ✅ CORRECT |
|----------|---------------|-----------|
| Stopping after round 3 because round 3 = 0 issues | 1 zero round is NOT early stop. You need 2 consecutive. | Continue to round 4. Only stop if round 3 AND round 4 are both 0. |
| Stopping after round 5 because round 3 and round 5 are both 0 | Rounds 3 and 5 are NOT consecutive — round 4 broke the streak. | Continue. Only stop when N and N+1 are both 0. |
| Stopping after round 2 with 0 issues because "looks clean" | Violates minimum 5 rounds AND lacks consecutive confirmation. | Continue to round 3 minimum. Even then, need round 2 AND round 3 both 0. |
| Declaring early stop after fixing issues from round 4 in round 5 | Round 5 is the fix round — you need to run ANOTHER review round (round 6) to confirm the fix is clean, then round 7 to confirm consecutive. | Fix → re-review (round 6) → if 0 issues → re-review again (round 7) → if still 0 → NOW early stop. |
| Counting L issues as "zero" | L issues are still issues. Zero means zero C/H/M/L. | Only I (info) or truly empty counts as a zero round. |

### Decision Flowchart

```
Round N completed with issues found?
├── YES → Fix all C/H/M issues → proceed to Round N+1
│         (Reset consecutive-zero counter to 0)
└── NO (0 issues or only I issues)
    └── Was Round N-1 also 0 issues?
        ├── YES → AND N >= 2 → ✅ EARLY STOP (2 consecutive zeros)
        └── NO  → Continue to Round N+1
                  (consecutive-zero counter = 1, need one more)
```

### Concrete Examples

**Example A — Cannot early stop (intermittent zeros):**
```
Round 1: H=1, M=2 → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → consecutive counter = 1 → continue (NOT early stop!)
Round 4: M=1      → Fix → counter RESET to 0 → continue
Round 5: 0 issues → counter = 1 → continue (NOT early stop!)
Round 6: 0 issues → counter = 2 → ✅ EARLY STOP (rounds 5 & 6 both 0)
```

**Example B — Cannot early stop (only 1 zero):**
```
Round 1: H=1      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → counter = 1 → continue (NOT early stop!)
Round 4: L=1      → Fix → counter RESET to 0 → continue
Round 5: 0 issues → counter = 1 → continue (NOT early stop!)
→ Minimum 5 rounds met, final round = 0 C/H/M → ✅ PASS GATE (but no early stop)
```

**Example C — Correct early stop:**
```
Round 1: M=2      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → counter = 1 → continue
Round 4: 0 issues → counter = 2 → ✅ EARLY STOP (rounds 3 & 4 both 0, consecutive)
```

## Gate Condition

Proceed to the next phase **ONLY IF**:
1. The Ralph loop has completed (minimum 5 rounds, or early stop met)
2. The **final round** has **zero C, H, and M issues** remaining
3. L and I issues may be carried forward into the next phase

## Design Review Checklist (Phases 1–3)

The reviewer must verify:
- [ ] Completeness: Does the deliverable cover all requirements from prior phases?
- [ ] Consistency: No contradictions with prior phase outputs
- [ ] Clarity: Unambiguous, explicit, no hand-waving
- [ ] Edge cases: Error paths, boundaries, and failure modes considered
- [ ] Feasibility: Can this actually be built and tested?
- [ ] Traceability: Every acceptance criterion maps to a test or design element

## Code Review Checklist (Phases 4–5)

The reviewer must verify:
- [ ] **Test quality**: Completeness, edge cases, descriptive names, one assertion per test where practical
- [ ] **Code quality**: Clean code, no duplication, proper abstractions, follows language idioms
- [ ] **TDD compliance**: No untested code; no code exists without a failing test justifying it
- [ ] **Phase 4 specific**: Are ALL tests genuinely failing? No premature implementation? No stubs that accidentally pass?
- [ ] **Phase 5 specific**: Is refactoring clean? Any regressions introduced? Is the minimum code principle respected?
- [ ] **Integration**: Do tests correctly import/reference the intended interfaces?

## Review Log Template

```
## Ralph Loop Review Log: Phase <N> — <Phase Name>

### Round 1
- C: 0 | H: 1 | M: 2 | L: 2 | I: 1
- Issues:
  - [H-1] ...
  - [M-1] ...
  - [M-2] ...
- Fixes applied: ...

### Round 2
- C: 0 | H: 0 | M: 1 | L: 1 | I: 0
- Issues:
  - [M-1] ...
- Fixes applied: ...

... (minimum 5 rounds) ...

### Final Gate
- Final round C/H/M count: 0
- Proceed to Phase <N+1>: YES / NO
```
