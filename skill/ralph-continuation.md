# Ralph Loop Continuation (Rounds 2+)

Load this file after Round 1 completes. Contains: main agent evaluation rules, stop conditions, decision flowchart, pre-stop articulation, and review checklists.

## Main Agent Critical Evaluation (主代理批判性审视)

⛔ **MUST NOT blindly adopt all suggestions.** Evaluate each against project context, then make an explicit decision:

| Decision   | Applicable To                   | Conditions                                                                                                                                                                   |
| ---------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ADOPT**  | All items                       | Default for C/H/M — apply the fix. P ADOPT is preferred but optional.                                                                                                        |
| **MODIFY** | All items                       | Suggestion has merit but needs adaptation — apply modified version, document deviation                                                                                       |
| **REJECT** | P/L/I items + Critical Opinions | Full discretion, just document rationale                                                                                                                                     |
| **REJECT** | C/H/M items                     | **Restricted** — only when reviewer's assumption about the project is factually incorrect. Must provide evidence. See `ralph-contested.md` for the contested issue protocol. |

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

## Known Issues Lifecycle

KI management is a **lifecycle concern** spanning the entire Ralph loop — not a single step.

### Recording

Before writing a new KI entry, **deduplicate** against existing KI entries: if a substantially similar finding already exists in the KI document, do NOT create a duplicate entry. Instead, update the existing entry's `re-raised-in` field with the current round number. Substantially similar = same location, same issue description (wording may differ but the underlying problem is the same).

Every finding NOT adopted this round (unadopted P, L/I, REJECTed P/L/I) MUST be recorded or deduplicated in the project's Known Issues document (e.g., `KnownIssues.md`). Each new entry: raised-in round, severity (with P/L tier), file location, description, why deferred, plan. Each deduplicated entry: append re-raised-in round number.

### Periodic Re-evaluation

Every 3 rounds (R3/R6/R9…), re-evaluate each KI entry: still real? fixed? severity changed?

⛔ KI re-evaluation is NOT complete until every entry has been assessed. Skip → counter does NOT increment. Stale KI → reviewer re-discovers already-fixed issues → stop condition never converges.

Inject surviving KIs into the next reviewer's context as `{KNOWN_ISSUES}` for independent assessment.

### Final Evaluation

At loop end (after STOP confirmed), perform a final KI evaluation:

- Summarize surviving entries
- Confirm triage accuracy
- Note any that should carry into the next phase

### KI vs New Findings

Findings already in the KI document are **known**, not **new**. Known findings excluded from the consecutive-zero counter. Contested C/H/M re-raises are always new.

## Rounds & Stop Conditions

The Ralph loop has **two exit paths**:

1. **Stop**: 2 consecutive rounds with zero **new** C/H/M findings (P/L do not reset counter; known issues excluded). No round cap.
2. **Persistent Issues Escalation**: If the same C/H/M findings recur across multiple rounds despite fixes, assess root cause — is this an implementation issue, or a prior-phase problem (unclear requirement, design flaw, test gap)? If prior-phase: escalate to user for clarification, or rollback per `ralph-review-loop.md` §Cross-Phase Escalation.

### ⛔ AVOID — Common Stop Condition Mistakes (READ CAREFULLY)

| ❌ WRONG                                                                 | ✅ CORRECT                                                                                                             |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Stopping after 1 zero round (need 2 **consecutive**)                    | Continue. Stop requires round N-1 AND round N both with 0 new C/H/M findings.                                         |
| Declaring stop immediately after fixing issues (fix round ≠ zero round) | Fix → next review round → reviewer confirms → then evaluate counter.                                                  |
| Counting new P or L findings as resetting the counter                   | P/L are quality-tier — triaged but do NOT reset. Only new C/H/M reset. Zero new C/H/M + any P/L → counter increments. |
| Counting re-discovered KI entries as "new findings"                     | Only findings NOT already in the KI document count as new. Known = excluded from counter.                             |
| Re-labeling H/M as "accepted design deviations" or P to avoid counting  | Use Contested Issue Protocol. Unresolved C/H/M block the gate.                                                        |
| Skipping KI re-evaluation at rounds divisible by 3                      | Counter does NOT increment unless KI re-evaluation is performed this round.                                           |

### Decision Flowchart

```
Evaluate after each round N, in this order (before starting round N+1):

1. Receive confirmed_findings from Precision Filter (output of dual-pass Steps A–C
   in ralph-review-loop.md §Review Process). If Steps A–C were not all completed
   this round (e.g. fact-gather was skipped), that round is invalid — re-run from
   Step B before proceeding here.
2. Main agent critical evaluation: ADOPT/REJECT/MODIFY each item
   (see §Main Agent Critical Evaluation above)
   - REJECT of C/H/M → contested issue → include in next round's context for reviewer
   - REJECTed C/H/M remain in tally until reviewer explicitly drops them
3. Apply all ADOPTed and MODIFYed C/H/M fixes
   ⛔ Steps 2 → 3 are sequential. Do not begin 3 before 2 is complete.
3a. ⛔ **Post-Fix Cross-Reference Consistency Scan** — before proceeding to 3b,
    verify each fix did not introduce contradictions with the rest of the document(s):
    a. **Identify touched concepts**: extract key terms, status symbols, format templates,
       and column names modified by each fix.
    b. **Scan for collisions**: grep all reviewed files for each touched concept.
       Check: does the fix contradict any untouched occurrence?
    c. **Verify template consistency**: if a fix changes a report format, definition, or
       template, verify all references to that format/template in the same document
       still align (e.g. Coverage Summary format vs "release-ready" definition).
    d. **Verify mirror tables**: if the review covers SKILL.md or any file with mirrored
       tables (marked "keep in sync"), verify the mirror is still consistent after fixes.
    e. **If contradiction found**: resolve before proceeding. Do NOT leave it for the next
       round to discover — that creates oscillation (fix → new contradiction → fix → ...).
    ⛔ This scan is MANDATORY. Skipping it is the #1 cause of oscillating review rounds.
3b. ⛔ Contested issue check — load `ralph-contested.md` if EITHER condition is true:
    a. Any C/H/M was REJECTed → include in next round's reviewer context
    b. Same C/H/M issue was downgraded (relabel to P/L/I or severity lowered)
       for the 2nd time → contested issue triggered
3c. Record unadopted findings to KI document (deduplicate first — see §Known Issues Lifecycle).
    Track downgraded C/H/M items for cross-round matching.
    ⛔ Steps 3 → 3b → 3c are sequential. Do not begin 3c before 3b is complete.
4.  ⛔ KI Re-evaluation Gate (rounds divisible by 3): counter does NOT increment
    unless KI re-evaluation is performed this round.
5.  GPAV: Submit gate tally to Watchdog via ralph_round_finding (if active)
    - Include ALL items in tally: ADOPTed, MODIFYed, AND contested
    - In dual-pass mode: submit confirmed findings only (not raw recall output)
6.  Classify findings: **new** (not in KI document) vs **known** (already in KI).
    Known findings excluded from counter. Contested C/H/M re-raises are always new.
7.  Count new C/H/M findings. If any new C/H/M remain → Reset consecutive-zero
    counter to 0 → Go to round N+1
8.  If zero new C/H/M but new P or L found:
    → P/L triage (ADOPT preferred but optional)
    → Increment consecutive-zero counter by 1 → Go to round N+1
9.  If zero new findings (only I, known re-findings, or nothing):
    → Increment consecutive-zero counter by 1
    → Counter = 1: Go to round N+1
    → Counter = 2: ⛔ DO NOT declare STOP yet. Proceed immediately to step 9b.

9b. ⛔ Pre-Stop Articulation Gate — BLOCKING, executes THIS round before any other action.
    ⛔ NOT deferred to the start of the next round.
    ⛔ STOP may NOT be declared until this block is produced and verdict is SAFE-TO-STOP.
    Output the following block verbatim (fill in brackets):

    === PRE-STOP ARTICULATION ===
    Rounds completed: [N]
    Dimensions covered: [what each round checked — one line per round]
    Complexity justification: [why 2 consecutive zero rounds is sufficient for THIS specific task]
    Most likely missed issue type: [what category could still be lurking]
    Remaining KI entries: [count] — triage accuracy: [ACCURATE / CONCERNS: ...]
    Verdict: SAFE-TO-STOP | CONCERNS-FOUND
    =============================

    → Verdict = SAFE-TO-STOP  →  ✅ STOP  →  proceed to step 11
    → Verdict = CONCERNS-FOUND  →  reset counter to 1  →  Go to round N+1
    → If CONCERNS-FOUND appears on 2 consecutive articulations
      →  reset counter to 0  →  Go to round N+1

10. Before starting round N+1, assess: are the same C/H/M findings recurring
    despite fixes?
    → If yes: evaluate root cause → ⛔ ESCALATE to user or ROLLBACK to prior phase
    → If findings are genuinely new each round → Go to round N+1
11. ⛔ User Approval Gate (only after ✅ STOP):
    ⛔ DO NOT ask user about phase transition before ✅ STOP.
    → User rejects → see current phase file §User Approval
```

### 🔒 Pre-Stop Articulation — ❌ AVOID Superficial Articulation

> Two consecutive zero rounds are the **minimum requirement**, not an upper bound. Stopping is "confirm that stopping is the correct decision" — not "execute when conditions are met."
> If articulation reveals concerns 2× in a row, reset counter to 0 (not 1).

- "Two rounds with zero issues, safe to stop" (no analysis of why zero implies safety)
- "Task is simple, two rounds is enough" (no specific justification)
- "Didn't find anything missing" (did not identify what type of issue could still be missed)
- "All remaining issues are in the KI document" (KI presence alone is not a stop argument — are they correctly triaged?)

## Review Checklists (Progressive Disclosure)

Review checklists are loaded per-phase from dedicated files:

| Phase            | Load this file     | Contains                                |
| ---------------- | ------------------ | --------------------------------------- |
| **1–3** (Design) | `review-design.md` | Design review checklist + Recall prompt |
| **4–5** (Code)   | `review-code.md`   | Code review checklist + Recall prompt   |

See `ralph-examples.md` for worked examples covering: intermittent zeros (A), L does not reset counter (B), correct stop (C), persistent issues escalation (D), contested issue lifecycle with graceful concession (E), contested issue via MODIFY — principled compromise (F), and contested issue escalation with 5-section dossier (G).
