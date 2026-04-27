# Ralph Loop Review Protocol

This protocol governs **all reviews** in the TDD Pipeline. It is invoked after every phase (1–5).

## When to Invoke

- **After Phases 1–3 (design phases)**: Launch Ralph loop **design review**
- **After Phases 4–5 (code phases)**: Launch Ralph loop **code review**

## Reviewer Selection

Spawn an **independent subagent** (oracle or dedicated reviewer) that was **not involved in creating the deliverable under review**. The reviewer has no bias from the creation process, but WILL be provided with prior phase outputs for cross-phase consistency checking.

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

```
for round N:
  present(deliverable, prior_phase_outputs) → reviewer
  review_against(gate_criteria)              # see phase-N-*.md files

  # Cross-phase escalation
  if root_cause in prior_phase:
    HALT loop
    ESCALATE to user: "Root cause in Phase N-k.
      Recommend rollback to Phase N-k, discard downstream, re-run Ralph loop."
    FORBIDDEN: fix prior-phase issues in current deliverable

  report: numbered issues with severity labels  # e.g. [H-1], [M-2], [L-3]
  tally: C=0, H=1, M=2, L=3, I=1
  fix: all C + H + M (L + I optional)
  log: { round: N, tally, fixes_applied }
```

## Rounds & Early Stop Rule

The Ralph loop has **three exit paths**:

1. **Early Stop**: 2 consecutive rounds with zero C/H/M/L issues (only I or nothing). Can trigger at any round N ≥ 2.
2. **Gate Pass**: At round N ≥ 5 with zero C/H/M in the current round (L acceptable). You MAY stop here; continuing to pursue early stop is optional.
3. **Max Rounds Escalation**: If C/H/M persist after 10 rounds, halt and escalate to the user with a summary.

**If early stop does NOT trigger**: you MUST complete at least 5 rounds before evaluating gate pass.

**Consecutive-zero counter**: Initialize at 0. Increment by 1 when a round has zero C/H/M/L. Reset to 0 on any round with C/H/M/L > 0. When counter = 2 → early stop.

### ⛔ AVOID — Common Early Stop Mistakes (READ CAREFULLY)

These are the most frequent errors LLMs make. **DO NOT do any of these:**

| ❌ WRONG | Why It's Wrong | ✅ CORRECT |
|----------|---------------|-----------|
| Stopping after round 3 because round 3 = 0 issues | 1 zero round is NOT early stop. You need 2 consecutive. | Continue to round 4. Only stop if round 3 AND round 4 are both 0. |
| Claiming early stop at round 5 because rounds 3 and 5 are both 0 | Rounds 3 and 5 are NOT consecutive — round 4 broke the streak. | Continue. Only stop when round N-1 and round N are both 0. Note: at N ≥ 5 with zero C/H/M, gate-pass is also available as an alternative to continuing. |
| Stopping after round 2 with 0 issues because "looks clean" (and round 1 was NOT 0) | Lacks consecutive confirmation — need both round 1 AND round 2 to be 0. (Early stop at round 2 IS valid if round 1 was also 0.) | Need round N-1 to also be 0. |
| Declaring early stop after fixing issues from round 4 | Round 5 reviews the fixed deliverable — if 0 issues, counter = 1 (round 4 was not zero). Need one more zero round. | Fix after round 4 → round 5 reviews → if 0 → round 6 reviews → if still 0 → early stop. |
| Counting L issues as "zero" | L issues are still issues. Zero means zero C/H/M/L. | Only I (info) or truly empty counts as a zero round. |

### Decision Flowchart

```
Evaluate after each round N, in this order (before starting round N+1):

1. Count non-I issues (C+H+M+L) found this round
2. If any C/H/M found → Fix all → Reset consecutive-zero counter to 0 → Go to round N+1
3. If only L found (no C/H/M):
   → L fix is optional → Reset consecutive-zero counter to 0
   → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue to round N+1 pursuing early stop)
   → If N < 5 → Go to round N+1
4. If zero issues (only I or nothing):
   → Increment consecutive-zero counter by 1
   → Counter = 2 → ✅ EARLY STOP
   → Counter = 1:
     → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue pursuing early stop)
     → Go to round N+1 (or stop if gate-pass is acceptable)
5. If round N+1 would exceed 10 → ⛔ MAX ROUNDS → Escalate to user
```

### Concrete Examples

**Example A — Cannot early stop (intermittent zeros):**
```
Round 1: H=1, M=2 → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → consecutive counter = 1 → continue (NOT early stop!)
Round 4: M=1      → Fix → counter RESET to 0 → continue
Round 5: 0 issues → counter = 1 → ✅ GATE PASS available (may stop here, or continue)
Round 6: 0 issues → counter = 2 → ✅ EARLY STOP (rounds 5 & 6 both 0)
```

**Example B — Cannot early stop (only 1 zero):**
```
Round 1: H=1      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → counter = 1 → continue (NOT early stop!)
Round 4: L=1      → Counter RESET to 0 (any non-I count resets; fixing L is optional) → continue
Round 5: 0 issues → counter = 1 → ✅ GATE PASS available (accept gate-pass now, or continue to pursue early stop)
=== Choosing gate-pass === ✅ PASS GATE
```

**Example C — Correct early stop (at round 4):**
```
Round 1: M=2      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 issues → counter = 1 → continue
Round 4: 0 issues → counter = 2 → ✅ EARLY STOP (rounds 3 & 4 both 0, consecutive)
```

**Example D — Escalation at max rounds:**
```
Round 1-7: persistent M issues, fixed and re-found → continue
Round 8: H=1 → Fix → continue
Round 9: M=1 → Fix → continue
Round 10: M=1 → ⛔ MAX ROUNDS → HALT → Escalate to user with issue summary
```

## Gate Condition

```
gate_proceed = ALL:
  ralph_termination IN [early_stop, gate_pass]  # NOT max_rounds
  final_round.C + .H + .M == 0                  # L/I acceptable, carried forward
```

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

... (continue until early stop or 5 rounds minimum) ...

### Final Gate
- Final round C/H/M count: 0
- Proceed to Phase <N+1>: YES / NO
```
