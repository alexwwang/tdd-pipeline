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

The Ralph loop has **two exit paths**:

1. **Stop**: 2 consecutive rounds where the reviewer finds zero **new** C/H/M/L issues (only I or nothing new). Can trigger at any round N ≥ 2. Previously deferred Known Issues do not count as new findings.
2. **Persistent Issues Escalation**: If the same C/H/M findings recur across multiple rounds despite fixes, this signals a prior-phase problem. Evaluate and choose:
   - **Escalate to user**: The issue requires clarification of requirements, constraints, or design intent that only the user can provide. Present a structured summary: what keeps recurring, what fixes were attempted, what information is missing.
   - **Rollback to prior phase**: The root cause is in a prior phase (requirements gap, design flaw, test gap). Follow the rollback protocol in `ralph-review-loop.md` §Cross-Phase Escalation.

**Stop triggers** when the consecutive-zero counter reaches 2 (zero new C/H/M/L findings for 2 rounds in a row). Counter resets to 0 on any round with new C/H/M/L > 0.

### ⛔ AVOID — Common Stop Condition Mistakes (READ CAREFULLY)

These are the most frequent errors LLMs make. **DO NOT do any of these:**

| ❌ WRONG | Why It's Wrong | ✅ CORRECT |
|----------|---------------|-----------|
| Stopping after round 3 because round 3 = 0 new findings | 1 zero round does NOT satisfy stop condition. You need 2 consecutive. | Continue to round 4. Only stop if round 3 AND round 4 both have 0 new findings. |
| Claiming stop at round 5 because rounds 3 and 5 both have 0 new findings | Rounds 3 and 5 are NOT consecutive — round 4 broke the streak. | Continue. Only stop when round N-1 and round N both have 0 new findings. |
| Stopping after round 2 with 0 new findings because "looks clean" (and round 1 had new findings) | Lacks consecutive confirmation — need both round 1 AND round 2 to have 0 new findings. | Need round N-1 to also have 0 new findings. |
| Declaring stop immediately after fixing issues | Fix round ≠ zero round. Next round reviews the fix — only if THAT round is also zero does the counter increment. | Fix → review fixed deliverable → evaluate counter on the review round. |
| Counting re-discovered KI entries as "new findings" | Issues already in the Known Issues document have been triaged — they are known, not new. | Only findings NOT already in the KI document count as new. |
| Declaring PASS after 1 zero round (not 2 consecutive) | Single zero does not satisfy stop condition. "PASS with conditions" is not PASS. | Continue. Stop requires round N-1 AND round N both with 0 new findings. |
| Counting new L findings as "effectively zero" | New L findings are still new findings. Zero new means zero new C/H/M/L. | Only I (info), re-discovered KIs, or truly empty = zero-new round. |
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
3b. **Document deferred findings**: For each L/I item and REJECTed L/I that is NOT fixed this round, record in the project's Known Issues document (e.g., `KnownIssues.md`). Each entry must include: raised-in round, severity, file location, description, why deferred, and plan. This ensures deferred observations are not lost across rounds and survive session boundaries.
4. **KI Re-evaluation** (Rounds 3+): Before spawning the next reviewer, re-evaluate each KI independently: is it still real? has it been incidentally fixed? should severity change? Remove false positives and document why. Inject surviving KIs into the next reviewer's context as `{KNOWN_ISSUES}` for independent assessment.
5. GPAV: Submit gate tally to Watchdog via ralph_round_finding (if active)
   - Include ALL items in tally: ADOPTed, MODIFYed, AND contested (REJECTED items still in tally)
   - In dual-pass mode: submit confirmed findings only (not raw recall output)
6. Classify findings: **new** (not in KI document) vs **known** (already in KI). Known findings excluded from counter. Contested C/H/M re-raises are always new.
7. Count new C/H/M/L findings. If any new C/H/M remain → Reset consecutive-zero counter to 0 → Go to round N+1
8. If only new L found (no new C/H/M):
   → L fix is optional → Reset consecutive-zero counter to 0 → Go to round N+1
9. If zero new findings (only I, known re-findings, or nothing):
   → Increment consecutive-zero counter by 1
   → Counter = 2:
     → 🔒 **必须完成停止前 Why Articulation**（见下方 §Pre-Stop Why Articulation）
     → If articulation confirms stopping is safe → ✅ STOP
     → If articulation reveals concerns → reset counter to 1 → Go to round N+1
   → Counter = 1:
     → Go to round N+1
10. Before starting round N+1, assess: are the same C/H/M findings recurring despite fixes?
    → If yes, this indicates a prior-phase root cause:
      → If the issue requires user clarification (ambiguous requirements, conflicting constraints) → ⛔ ESCALATE to user with structured summary
      → If the root cause is in a prior phase (missing requirement, design flaw, test gap) → ⛔ ROLLBACK to that phase per `ralph-review-loop.md` §Cross-Phase Escalation
    → If findings are genuinely new each round (different issues, not recurrence) → Go to round N+1
```

### 🔒 Pre-Stop Why Articulation (MUST complete when consecutive-zero counter reaches 2)

Before confirming stop, articulate: rounds completed, dimensions covered per round, task complexity justification, what issue type is most likely to be missed, and how many known issues remain in the KI document. If you cannot clearly articulate this, continue to the next round.

> ⚠️ Two consecutive zero-new-finding rounds are the **minimum requirement**, not an upper bound. Stopping is not "execute when conditions are met" — it is "confirm that stopping is the correct decision."
> If articulation reveals concerns 2 times in a row, reset counter to 0 (not 1).

**❌ What superficial pre-stop articulation looks like**
- "Two rounds with zero issues, safe to stop" (no analysis of why zero new findings implies safety)
- "Task is simple, two rounds is enough" (no specific justification for this judgment)
- "Didn't find anything missing" (did not identify what type of issue could still be missed)
- "All remaining issues are in the KI document" (KI presence alone is not a stop argument — are they correctly triaged?)

## Review Checklists (Progressive Disclosure)

Review checklists are loaded per-phase from dedicated files:

| Phase | Load this file | Contains |
|-------|---------------|----------|
| **1–3** (Design) | `review-design.md` | Design review checklist + Recall prompt |
| **4–5** (Code) | `review-code.md` | Code review checklist + Recall prompt |

See `ralph-examples.md` for worked examples covering: intermittent zeros (A), single zero (B), correct stop (C), persistent issues escalation (D), contested issue lifecycle with graceful concession (E), contested issue via MODIFY — principled compromise (F), and contested issue escalation with 5-section dossier (G).
