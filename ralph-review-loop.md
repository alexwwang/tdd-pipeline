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
  present(deliverable, prior_phase_outputs, contested_issues_from_prior_rounds) → reviewer
  review_against(gate_criteria)              # see phase-N-*.md files

  # Cross-phase escalation (UNCONDITIONAL — cannot be overridden by critical evaluation)
  if root_cause in prior_phase:
    HALT loop
    ESCALATE to user: "Root cause in Phase N-k.
      Recommend rollback to Phase N-k, discard downstream, re-run Ralph loop."
    FORBIDDEN: fix prior-phase issues in current deliverable

  # Step 1: Reviewer produces three output categories (see §Reviewer Output Requirements)
  report:
    severity_issues: numbered with C/H/M/L/I labels
      # e.g.: [H-1] Missing boundary condition handling in AC-3
    constructive_suggestions: actionable fixes paired with severity issues
    critical_opinions: architectural/strategic critique (when substantive concerns exist)

  tally: C=0, H=1, M=2, L=3, I=1

  # Step 2: Main agent critical evaluation (see §Main Agent Critical Evaluation)
  for each review item:
    evaluate_against(project_context) → ADOPT | REJECT | MODIFY
    # C/H/M issues: REJECT requires documented evidence of reviewer assumption error
    # L/I/opinions: full discretion

  # Step 3: Apply fixes
  fix: all ADOPTed and MODIFYed C + H + M (L + I + ADOPTed opinions optional)
  # REJECTed C/H/M remain in tally — they still count toward gate condition
  # unless reviewer explicitly drops them in the next round (see §Contested Issue Protocol)

  log: { round: N, tally, contested, evaluation_decisions, fixes_applied }
```

> **Note**: The pseudocode above is a simplified overview. Full details for each step are in the sections below.

### Reviewer Output Requirements

The reviewer produces **three output categories** per round. Their relationship is:

| Category | Describes | Maps to Severity Issues |
|----------|-----------|------------------------|
| **Severity-tagged Issues** | *Defects* — what is wrong | Each classified C/H/M/L/I |
| **Constructive Suggestions** | *Fixes* — how to resolve defects | Paired 1:1 with C/H/M issues; optional for L; **not required for I** |
| **Critical Opinions** | *Strategic concerns* — whether the approach is right | Independent of specific defects; optional — provide only when substantive concerns exist |

**In rounds with zero C/H/M issues**: Constructive Suggestions and Critical Opinions are both **optional**. Exempt rounds from mandatory output to avoid degenerate/performative content.

#### 1. Constructive Suggestions (建设性建议)

Actionable, specific recommendations for improving the deliverable. Required for every C/H/M issue; optional for L; not required for I.

Each suggestion MUST include:
   - What to change and where
   - Why the change improves the deliverable
   - A concrete example or reference implementation when applicable
   - No vague advice like "consider improving X" — every suggestion must be directly actionable

#### 2. Critical Opinions (批判性意见)

Substantive critique that challenges assumptions, identifies risks, or questions design decisions. **Provide only when the reviewer has genuine, substantive concerns** — do not manufacture critique to fill a template.

When provided, Critical Opinions MUST:
   - Challenge the reasoning behind choices, not just surface-level symptoms
   - Identify potential failure modes, edge cases, or blind spots the author may have missed
   - Question whether the current approach is the RIGHT approach (not just whether it works)
   - Highlight trade-offs being made and whether they are acceptable given project constraints
   - Be evidence-based — cite specific sections, requirements, or prior-phase outputs that conflict

### Main Agent Critical Evaluation (主代理批判性审视)

Upon receiving the reviewer's report (issues, constructive suggestions, and critical opinions), the **main agent MUST NOT blindly adopt all suggestions**. Instead, apply a structured critical evaluation:

1. **Read the full reviewer report** — understand the reasoning behind each suggestion and opinion, not just the surface-level recommendation.

2. **Evaluate against project context** — consider:
   - Project's actual constraints (timeline, tech stack, team skills, existing architecture)
   - Whether the suggestion addresses a real problem or a theoretical one
   - Whether the "improvement" introduces new complexity or risks that outweigh the benefit
   - Whether the reviewer's assumptions about the project are correct

3. **Make an explicit ADOPT/REJECT/MODIFY decision** for each review item:

   | Decision | Applicable To | Conditions |
   |----------|--------------|------------|
   | **ADOPT** | All items | Default for C/H/M — apply the fix |
   | **MODIFY** | All items | Suggestion has merit but needs adaptation — apply modified version, document deviation |
   | **REJECT** | L/I items + Critical Opinions | Full discretion, just document rationale |
   | **REJECT** | C/H/M items | **Restricted** — only when reviewer's assumption about the project is factually incorrect. Must provide evidence. See §Contested Issue Protocol below. |

4. **Document the evaluation** — for each non-trivial suggestion/opinion, record in the evaluation table (see Review Log Template).

#### REJECT Rules for C/H/M Issues

The severity system is the quality gate: C/H/M means **must fix**. REJECT of a C/H/M is an **exception**, not a routine action. It is only valid when:

- The reviewer's assumption about the project requirements, constraints, or prior-phase outputs is factually wrong
- The reviewer identified a "defect" that is actually intended behavior documented elsewhere

**Invalid REJECT reasons** (these will not pass gate):
- "We don't have time" → not a valid reason to skip a must-fix issue
- "I disagree with the priority" → severity is the reviewer's call, not the author's
- "It works in practice" → if the reviewer identified a risk, it needs addressing

#### Contested Issue Protocol

When the main agent REJECTs a C/H/M issue, it becomes a **contested issue**:

1. The main agent documents the REJECTION rationale in the review log
2. The contested issue is included in the next round's context for the reviewer
3. The reviewer in the next round MUST explicitly address each contested issue:
   - **Accept the rejection** (reviewer agrees the assumption was wrong) → issue is dropped from tally
   - **Provide additional evidence** to re-raise the issue → issue stays in tally; main agent should ADOPT or MODIFY based on the new evidence. If the main agent REJECTs again despite additional evidence, this counts as a second dispute round and triggers the escalation rule (max 2 dispute rounds before user escalation).
4. **Escalation rule**: If the same issue remains contested after 2 rounds, **escalate to the user** for resolution. Do NOT silently drop or keep contested issues beyond 2 rounds.

**Key invariant**: REJECTed C/H/M issues **remain in the gate tally** until the reviewer explicitly drops them. The gate condition `final_round.C + .H + .M == 0` uses the tally after contested-issue resolution, not before.

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

1. Receive reviewer report (severity issues + constructive suggestions + critical opinions)
2. Main agent critical evaluation: ADOPT/REJECT/MODIFY each item (see §Main Agent Critical Evaluation)
   - REJECT of C/H/M → contested issue → include in next round's context for reviewer
   - REJECTed C/H/M remain in tally until reviewer explicitly drops them
3. Apply all ADOPTed and MODIFYed C/H/M fixes
4. Count non-I issues (C+H+M+L) in the tally (after contested resolution from THIS round)
5. If any C/H/M remain in tally → Reset consecutive-zero counter to 0 → Go to round N+1
6. If only L found (no C/H/M):
   → L fix is optional → Reset consecutive-zero counter to 0
   → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue to round N+1 pursuing early stop)
   → If N < 5 → Go to round N+1
7. If zero issues (only I or nothing):
   → Increment consecutive-zero counter by 1
   → Counter = 2 → ✅ EARLY STOP
   → Counter = 1:
     → If N ≥ 5 → ✅ GATE PASS available (you MAY stop here; or continue pursuing early stop)
     → Go to round N+1 (or stop if gate-pass is acceptable)
8. If round N+1 would exceed 10 → ⛔ MAX ROUNDS → Escalate to user
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

**Example E — Contested issue lifecycle (reviewer re-raises, then drops):**
```
Round 3: reviewer finds [M-3] Missing timeout handling in API client
  → main agent REJECTs: "Timeout is handled by the gateway, not the client (see Phase 2 §Component Gateway, line 42)"
  → [M-3] becomes contested, forwarded to Round 4
  → Tally still includes [M-3] in M count

Round 4: reviewer receives contested [M-3] in context
  → reviewer checks Phase 2 Gateway component → confirms timeout IS handled there
  → reviewer ACCEPTS rejection → [M-3] dropped from tally
  → No new C/H/M issues → tally = 0
  → Consecutive-zero counter = 1 → continue
```

**Example F — Contested issue escalation (2-round dispute):**
```
Round 5: reviewer finds [H-2] No input validation on user email field
  → main agent REJECTs: "Email validation is the frontend's responsibility per Phase 1 AC-3"
  → [H-2] contested, forwarded to Round 6

Round 6: reviewer re-examines Phase 1 AC-3
  → reviewer provides evidence: "AC-3 says 'system must reject invalid emails' — 'system' includes backend (see Phase 1 US-1 scope: 'server-side validation')"
  → reviewer re-raises [H-2] with additional evidence
  → main agent must now ADOPT or MODIFY (cannot REJECT again on same grounds)
  → main agent MODIFYs: "Add validation but only as defense-in-depth, not primary validation"

Round 7: reviewer verifies the modified fix
  → No new issues → tally = 0
```

**Example G — Contested issue escalation to user (unresolvable):**
```
Round 3: reviewer finds [H-1] Security vulnerability in auth token handling
  → main agent REJECTs: "This follows the standard OAuth2 flow per our architecture"
  → [H-1] contested, forwarded to Round 4

Round 4: reviewer re-raises with evidence
  → main agent REJECTs again (2nd round of dispute)
  → ⚠️ 2-round dispute limit reached → ESCALATE to user
  → HALT loop, present both positions to user for resolution
```

## Gate Condition

```
gate_proceed = ALL:
  ralph_termination IN [early_stop, gate_pass]  # NOT max_rounds
  final_round.C + .H + .M == 0                  # L/I acceptable, carried forward
```

## Design Review Checklist (Phases 1–3)

The reviewer must verify each item below. When defects are found, provide constructive suggestions per §Reviewer Output Requirements. When substantive strategic concerns exist, provide critical opinions.

- [ ] Completeness: Does the deliverable cover all requirements from prior phases?
- [ ] Consistency: No contradictions with prior phase outputs
- [ ] Clarity: Unambiguous, explicit, no hand-waving
- [ ] Edge cases: Error paths, boundaries, and failure modes considered
- [ ] Feasibility: Can this actually be built and tested?
- [ ] Traceability: Every acceptance criterion maps to a test or design element

## Code Review Checklist (Phases 4–5)

The reviewer must verify each item below. When defects are found, provide constructive suggestions per §Reviewer Output Requirements. When substantive strategic concerns exist, provide critical opinions.

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
- Constructive Suggestions:
  - <paired with C/H/M issues; optional for L; not required for I>
- Critical Opinions (if any):
  - <substantive critique, or omit if no genuine concerns>
- Main Agent Critical Evaluation:

  | Item | Decision | Rationale | Action |
  |------|----------|-----------|--------|
  | [H-1] | ADOPT | ... | Fixed: ... |
  | [M-1] | MODIFY | ... | Adapted: ... |
  | [M-2] | REJECT | ... | No change (contested — see next round) |

- Fixes applied: ...
- Contested issues forwarded to next round:

  | Issue | First contested | Dispute round | Reviewer action needed |
  |-------|----------------|---------------|----------------------|
  | [M-2] | Round 1 | 1 of 2 | Accept rejection OR re-raise with evidence |

### Round 2
- Contested from prior round:
  - [M-2] (disputed Round 1): Reviewer response → <ACCEPT rejection / RE-RAISE with evidence: ...>
  - Resolution: <dropped from tally / remains in tally>
- C: 0 | H: 0 | M: 1 | L: 1 | I: 0
- Issues:
  - [M-1] ...
- Constructive Suggestions:
  - ...
- Critical Opinions:
  - (omitted — no substantive concerns this round)
- Main Agent Critical Evaluation:

  | Item | Decision | Rationale | Action |
  |------|----------|-----------|--------|
  | [M-1] | ADOPT | ... | Fixed: ... |

- Fixes applied: ...
- Contested issues forwarded to next round: (none)

... (continue until early stop or 5 rounds minimum) ...

### Final Gate
- Final round C/H/M count (after contested resolution): 0
- Proceed to Phase <N+1>: YES / NO
```
