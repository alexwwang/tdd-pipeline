# Ralph Loop Continuation (Rounds 2+)

Load this file after Round 1 completes. Contains: main agent evaluation rules, stop conditions, decision flowchart, pre-stop articulation, and review checklists.

## Main Agent Critical Evaluation (主代理批判性审视)

⛔ **MUST NOT blindly adopt all suggestions.** Evaluate each against project context, then make an explicit decision:

| Decision | Applicable To | Conditions |
|----------|--------------|------------|
| **ADOPT** | All items | Default for C/H/M — apply the fix |
| **MODIFY** | All items | Suggestion has merit but needs adaptation — apply modified version, document deviation |
| **REJECT** | L/I items + Critical Opinions | Full discretion, just document rationale |
| **REJECT** | C/H/M items | **Restricted** — only when reviewer's assumption about the project is factually incorrect. Must provide evidence. See `ralph-contested.md` for the contested issue protocol. |

### REJECT Rules for C/H/M Issues

C/H/M means **must fix**. REJECT is an **exception**, valid only when:
- Reviewer's assumption about requirements, constraints, or prior-phase outputs is factually wrong
- Reviewer identified a "defect" that is actually intended behavior documented elsewhere

**Invalid REJECT reasons** (will not pass gate):
- "We don't have time" / "I disagree with the priority" / "It works in practice"

### ❌ What wrong evaluation looks like

- **Mechanical adoption**: applying all suggestions without independent judgment
- **Dismissive rejection**: characterizing H/M issues as "known design deviations" without evidence — severity manipulation
- **Fix-and-declare**: declaring gate PASS after fixing without another review round — fixes may introduce regressions
- **Deferred escaping**: pushing C/H/M to "handle in a later phase" — each gate must close on its own

## Rounds & Stop Conditions

The Ralph loop has **three exit paths**:

1. **Stop**: 2 consecutive rounds with zero C/H/M/L issues (only I or nothing). Can trigger at any round N ≥ 2.
2. **Gate Pass**: At round N ≥ 5 with zero C/H/M in the current round (L acceptable). You MAY stop here; continuing to pursue stop is optional.
3. **Max Rounds Escalation**: If C/H/M persist after 10 rounds, halt and escalate to the user with a summary.

**If stop does NOT trigger**: you MUST complete at least 5 rounds before evaluating gate pass.

**Consecutive-zero counter**: Initialize at 0. Increment by 1 when a round has zero C/H/M/L. Reset to 0 on any round with C/H/M/L > 0. When counter = 2 → 必须完成 Pre-Stop Why Articulation → 确认安全后 stop。

### ⛔ AVOID — Common Stop Condition Mistakes (READ CAREFULLY)

These are the most frequent errors LLMs make. **DO NOT do any of these:**

| ❌ WRONG | Why It's Wrong | ✅ CORRECT |
|----------|---------------|-----------|
| Stopping after round 3 because round 3 = 0 issues | 1 zero round does NOT satisfy stop condition. You need 2 consecutive. | Continue to round 4. Only stop if round 3 AND round 4 are both 0. |
| Claiming stop at round 5 because rounds 3 and 5 are both 0 | Rounds 3 and 5 are NOT consecutive — round 4 broke the streak. | Continue. Only stop when round N-1 and round N are both 0. Note: at N ≥ 5 with zero C/H/M, gate-pass is also available as an alternative to continuing. |
| Stopping after round 2 with 0 issues because "looks clean" (and round 1 was NOT 0) | Lacks consecutive confirmation — need both round 1 AND round 2 to be 0. (Early stop at round 2 IS valid if round 1 was also 0.) | Need round N-1 to also be 0. |
| Declaring stop after fixing issues from round 4 | Round 5 reviews the fixed deliverable — if 0 issues, counter = 1 (round 4 was not zero). Need one more zero round. | Fix after round 4 → round 5 reviews → if 0 → round 6 reviews → if still 0 → stop. |
| Counting L issues as "zero" | L issues are still issues. Zero means zero C/H/M/L. | Only I (info) or truly empty counts as a zero round. |
| Declaring PASS after 1 zero round (not 2 consecutive) | Single zero does not satisfy stop condition. "PASS with conditions" is not PASS. | Continue. Stop requires round N-1 AND round N both zero. |
| Counting 0C/0H as "zero" while L>0 exists | Stop condition requires zero C/H/M/L, not zero C/H only. L still resets the counter. | L issues count. Only I or truly empty = zero round. |
| Re-labeling H/M issues as "accepted design deviations" to avoid counting | Downgrading severity without the contested issue protocol is manipulation, not judgment. | Use Contested Issue Protocol. Unresolved C/H/M block the gate. |
| Declaring PASS after fixing issues without another review round | Fix-and-declare is not review — fixes may introduce regressions. | Fix → next review round → reviewer confirms → then evaluate stop. |

### Decision Flowchart

```
Evaluate after each round N, in this order (before starting round N+1):

1. Receive reviewer report (severity issues + constructive suggestions + critical opinions)
2. Main agent critical evaluation: ADOPT/REJECT/MODIFY each item (see §Main Agent Critical Evaluation)
   - REJECT of C/H/M → contested issue → include in next round's context for reviewer
   - REJECTed C/H/M remain in tally until reviewer explicitly drops them
3. Apply all ADOPTed and MODIFYed C/H/M fixes
4. GPAV: Submit gate tally to Watchdog via ralph_round_finding (if active)
   - Include ALL items in tally: ADOPTed, MODIFYed, AND contested (REJECTED items still in tally)
   - In dual-pass mode: submit confirmed findings only (not raw recall output)
5. Count non-I issues (C+H+M+L) in the tally (after contested resolution from THIS round)
6. If any C/H/M remain in tally → Reset consecutive-zero counter to 0 → Go to round N+1
7. If only L found (no C/H/M):
   → L fix is optional → Reset consecutive-zero counter to 0
   → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue to round N+1 pursuing stop)
   → If N < 5 → Go to round N+1
8. If zero issues (only I or nothing):
   → Increment consecutive-zero counter by 1
   → Counter = 2:
     → 🔒 **必须完成停止前 Why Articulation**（见下方 §Pre-Stop Why Articulation）
     → If articulation confirms stopping is safe → ✅ EARLY STOP
     → If articulation reveals concerns → reset counter to 1 → Go to round N+1
   → Counter = 1:
     → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue pursuing stop)
      → Go to round N+1 (or stop if gate-pass is acceptable)
9. If round N+1 would exceed 10 → ⛔ MAX ROUNDS → Escalate to user
```

### 🔒 Pre-Stop Why Articulation (MUST complete when consecutive-zero counter reaches 2)

Before confirming stop, articulate: rounds completed, dimensions covered per round, task complexity justification, and what issue type is most likely to be missed. If you cannot clearly articulate this, continue to the next round.

> ⚠️ Two consecutive zero rounds are the **minimum requirement**, not an upper bound. Early stop is not "execute when conditions are met" — it is "confirm that stopping is the correct decision."
> If articulation reveals concerns 2 times in a row, reset counter to 0 (not 1).

**❌ What superficial pre-early-stop articulation looks like**
- "Two rounds with zero issues, safe to stop" (no analysis of why zero issues implies safety)
- "Task is simple, two rounds is enough" (no specific justification for this judgment)
- "Didn't find anything missing" (did not identify what type of issue could still be missed)

## Review Checklists (Progressive Disclosure)

Review checklists are loaded per-phase from dedicated files:

| Phase | Load this file | Contains |
|-------|---------------|----------|
| **1–3** (Design) | `review-design.md` | Design review checklist + Recall prompt |
| **4–5** (Code) | `review-code.md` | Code review checklist + Recall prompt |

See `ralph-examples.md` for worked examples covering: intermittent zeros (A), single zero (B), correct stop (C), max rounds escalation (D), contested issue lifecycle with graceful concession (E), contested issue via MODIFY — principled compromise (F), and contested issue escalation with 5-section dossier (G).
