# Ralph Loop Review Protocol

Phases 1–5 only. Phase 6 uses `phase-6-pre-release-testing.md` Part 5.

## When to Invoke

- **After Phases 1–3 (design phases)**: Launch Ralph loop **design review**
- **After Phases 4–5 (code phases)**: Launch Ralph loop **code review**

## Reviewer Selection

Spawn an **independent subagent** not involved in creating the deliverable. Provide prior phase outputs for cross-phase consistency.

**Reviewer prompt MUST NOT contain**: stop conditions, cumulative tallies, prior round findings, fix lists, round counts, or loop state. Principle: reviewer evaluates only the current deliverable; past/future loop state creates anchoring bias.

**Reviewer prompt MUST contain**: a `[REVIEW SCOPE]` block derived from the current phase file. This is NOT a scope-limiting hint — it is a phase boundary declaration that prevents cross-phase noise. See §Review Scope Declaration below.

⛔ **Before dispatching ANY reviewer subagent (Recall or Precision), verify prompt contains NONE of**:
- round number ("Round N", "第N轮", "N of 10")
- prior findings ("R1 found X", "上轮发现了")
- cumulative totals ("total N issues so far", "累计N个问题")
- fix status ("all issues resolved", "已全部修复")
- stop-condition hints ("if clean you may stop", "可提前结束", "looks good overall")

## Review Scope Declaration

Every reviewer subagent prompt MUST open with a `[REVIEW SCOPE]` block. This block
is populated from the current phase file's Gate checklist — it is not ad-hoc.

**Structure**:
```
[REVIEW SCOPE — PHASE <N>: <Phase Name>]
IN SCOPE: <items drawn verbatim from the Gate checklist of the current phase file>
OUT OF SCOPE (do not raise as C/H/M — log as DEFERRED if noted at all):
  - Implementation details → Phase 3/4
  - Test execution results → Phase 3
  - Performance optimization → Phase 5
  - Any concern whose resolution requires output from a later phase
DEFERRED items: record in findings list with tag [DEFERRED], severity I.
                They do NOT enter the fix loop and do NOT count toward stop condition.
```

**Rule**: A finding is IN SCOPE if and only if it can be evaluated and fixed
using only the current phase's deliverable and prior-phase outputs already available.
If fixing the finding requires waiting for a later phase's output → OUT OF SCOPE → DEFERRED.

⛔ This block is scope declaration, not anchoring. It does not reference prior round
findings, fix status, or cumulative tallies — those remain forbidden.

## Severity Classification

Three tiers with six levels:

### Defect Tier (counted in stop condition)

| Sev | Name     | Definition                                                                            |
| --- | -------- | ------------------------------------------------------------------------------------- |
| C   | Critical | Fundamental flaw; deliverable is wrong, dangerous, or useless                         |
| H   | High     | Significant gap or serious risk                                                       |
| M   | Major    | Behavioral defect: code does something different from interface/documentation promise |

**Heuristic**: "If I fix this, will runtime behavior change?" → Yes → Defect Tier.

Failure-mode split: when one code fact creates multiple risks, report distinct lifecycle/resource, capacity/bounds, correctness/concurrency, security/isolation, and testability/design concerns separately with independent severities.

Context boundary: distinguish current requirement violations from future scalability concerns; future-only risks are I/P unless they break stated requirements.

Anti-patterns: labeling a refactoring suggestion as M; labeling a real defect as P; merging different severities because they share one location.

### Quality Tier (NOT counted)

| Sev | Name     | Definition                                           |
| --- | -------- | ---------------------------------------------------- |
| P   | Proposal | Cross-file/architectural restructuring               |
| L   | Low      | Single-location improvement (rename, comment, style) |

### Observation Tier

| Sev | Name | Definition                             |
| --- | ---- | -------------------------------------- |
| I   | Info | Observation or question with no defect |

Legacy mapping: `severity-migration.md`

## Review Process

```
for round N:

  # ── STEP A: Recall Pass ──────────────────────────────────────────────────────
  # Load review-design.md (Phase 1–3) OR review-code.md (Phase 4–5).
  # ⛔ PROMPT CONTAMINATION CHECK — output this checklist and confirm each item
  #    is ABSENT from the Recall subagent prompt before dispatching:
  #
  #    [ ] round number or round count (e.g. "Round 2", "第2轮", "N of 10")
  #    [ ] prior round findings (e.g. "R1 found X", "上轮发现了")
  #    [ ] cumulative tallies (e.g. "total 5 issues so far", "累计5个问题")
  #    [ ] fix status (e.g. "all issues resolved", "已全部修复")
  #    [ ] stop-condition hints (e.g. "if clean you may stop", "可提前结束")
  #
  # ⛔ SCOPE PRESENCE CHECK — output this checklist and confirm each item
  #    is PRESENT in the Recall subagent prompt before dispatching:
  #
  #    [ ] [REVIEW SCOPE] block derived from current phase Gate checklist
  #    [ ] IN SCOPE items populated
  #    [ ] OUT OF SCOPE items populated
  #
  # ⛔ Only after confirming ALL FIVE absence items AND ALL THREE presence items
  #    may you dispatch Recall.
  # Dispatch: deliverable + prior_phase_outputs + contested_issues → Recall subagent
  recall_output ← Recall subagent

  # ── STEP B: Fact-Gathering (main agent — NOT delegatable to subagent) ────────
  # ⛔ This step is performed by the MAIN AGENT, not a subagent.
  # ⛔ Do NOT dispatch Precision subagent until VERIFIED_FACTS is non-empty.
  # Use facts_to_gather from review-design.md OR review-code.md §Fact-Gathering.
  # Produce visible output: VERIFIED_FACTS = { <finding_id>: <raw_evidence> }
  # If VERIFIED_FACTS is empty after attempting fact-gather → HALT, diagnose blocker.
  #
  # ⛔⛔⛔ FACT-GATHER OUTPUT FORMAT — RAW FACTS ONLY ⛔⛔⛔
  # Fact-gather MUST produce ONLY raw evidence (exact quotes, line numbers,
  # structural observations). It MUST NOT contain:
  #   - CONFIRM / DOWNGRADE / REJECT verdicts
  #   - Main agent's opinion on whether a finding is valid
  #   - Severity reclassification suggestions
  #   - "Pre-evaluation" or "pre-judgment" of any kind
  #
  # WHY: The Precision subagent's ENTIRE VALUE is independent judgment.
  # If main agent pre-judges findings during fact-gather, the Precision
  # subagent receives biased input → independence destroyed → dual-pass
  # degrades to single-pass with extra steps.
  #
  # CORRECT:  "C-001: Line 46 writes 're-run with flags'. Line 102 writes 'Do NOT initiate re-runs'."
  # WRONG:    "C-001: CONFIRM — lines 46 and 102 contradict"
  VERIFIED_FACTS ← main_agent_fact_gather(recall_output)

  # ⛔ MANDATORY OUTPUT — emit the following block verbatim before proceeding to Step C.
  # Step C MUST NOT begin until this block appears in the output.
  #
  #   [FACT-GATHER COMPLETE]
  #   findings_received: <N from Recall output>
  #   facts_gathered: <M entries in VERIFIED_FACTS>
  #   VERIFIED_FACTS = {
  #     <finding_id>: <raw evidence — exact quote or line reference>
  #     ...
  #   }
  #   blocker: NONE | <reason if facts_gathered = 0>
  #
  # Validity rules (self-check before continuing):
  #   - findings_received > 0 and facts_gathered > 0  → proceed to Step C
  #   - findings_received > 0 and facts_gathered = 0  → emit blocker, HALT
  #   - findings_received = 0                         → emit blocker, HALT
  #
  # ⛔ facts_gathered = 0 while findings_received > 0 is a protocol violation.
  # ⛔ Emitting CONFIRM/DOWNGRADE/REJECT inside VERIFIED_FACTS is a protocol violation.

  # ── STEP C: Precision Filter ──────────────────────────────────────────────────
  # Load review-precision-filter.md. Inject VERIFIED_FACTS + recall_output.
  # ⛔ Same prompt contamination rules as Step A apply to the Precision prompt too.
  # ⛔ Same scope presence rules as Step A apply to the Precision prompt too.
  #    [REVIEW SCOPE] block must be forwarded from Recall into Precision unchanged.
  #
  # ⛔⛔⛔ PRECISION MUST RUN AS INDEPENDENT SUBAGENT ⛔⛔⛔
  # The Precision pass MUST be dispatched as a separate subagent (oracle or
  # equivalent). The main agent MUST NOT:
  #   - Execute Precision judgments itself (no CONFIRM/DOWNGRADE/REJECT by main)
  #   - Pre-filter which findings to send (send ALL findings, always)
  #   - Pre-sort findings by expected outcome
  #   - Include its own pre-evaluation in the Precision prompt
  #
  # The Precision prompt MUST contain:
  #   1. ALL raw Recall findings (complete list, no omissions)
  #   2. VERIFIED_FACTS (raw evidence only, no verdicts)
  #   3. The Precision filter instructions (from review-precision-filter.md)
  # The Precision prompt MUST NOT contain:
  #   1. Main agent's pre-judgment on any finding
  #   2. "Suggested" or "recommended" verdicts
  #   3. Hints about which findings are "probably false positives"
  #
  # WHY: Main agent has context bias (it built the recall prompt, gathered facts,
  # and formed opinions). Independent Precision breaks this bias loop.
  # If main agent = Precision agent, dual-pass collapses to echo chamber.
  confirmed_findings ← Precision subagent(VERIFIED_FACTS, recall_output)

  # ── STEP D: Evaluate → Fix → Scan → Log  (SEQUENTIAL: each step must complete
  #                                                     before the next begins) ──
  # D1: main_agent evaluates each confirmed finding → ADOPT | REJECT | MODIFY
  #     (see ralph-continuation.md §Main Agent Critical Evaluation)
  # D2: fix all ADOPTed/MODIFYed C/H/M (P/L/I/ADOPTed_opinions optional)
  # D2.5: ⛔ Post-Fix Cross-Reference Consistency Scan (MANDATORY)
  #       For each fix: identify touched concepts → grep all reviewed files →
  #       verify no contradictions with untouched content → verify mirror tables.
  #       If contradiction found: resolve before proceeding.
  #       See ralph-continuation.md §Step 3a for full protocol.
  #       This is the #1 defense against oscillating review rounds.
  # D3: log { round, tally, contested, evaluation_decisions, fixes_applied }
  # ⛔ Do NOT begin D2 before D1 is complete.
  # ⛔ Do NOT begin D2.5 before D2 is complete.
  # ⛔ Do NOT begin D3 before D2.5 is complete.
  main_agent_evaluate_and_fix(confirmed_findings)
  log: { round, tally, contested, evaluation_decisions, fixes_applied }

  # ⛔ MANDATORY ROUND CLOSE — emit the following block as the FINAL output of every round,
  # after the log entry and before any other action.
  #
  #   [ROUND <N> CLOSE]
  #   new_C: <n>   new_H: <n>   new_M: <n>
  #   cumulative_open_CHM: <n>
  #   consecutive_zero_CHM_rounds: <0|1|2>
  #   gate_proceed: NO | YES
  #   next_action: CONTINUE_LOOP | STOP_LOOP
  #
  # Validity rules (self-check before emitting next_action):
  #   - gate_proceed = YES   requires consecutive_zero_CHM_rounds = 2  → next_action = STOP_LOOP
  #   - gate_proceed = NO    regardless of any other condition          → next_action = CONTINUE_LOOP
  #
  # ⛔ next_action = STOP_LOOP is INVALID unless consecutive_zero_CHM_rounds = 2.
  # ⛔ Emitting next_action = STOP_LOOP when gate_proceed = NO is a protocol violation.
  # ⛔ Omitting this block entirely is a protocol violation equivalent to fabricating gate_proceed = YES.
  #
  # If next_action = CONTINUE_LOOP → load ralph-continuation.md for the next round.
  # If next_action = STOP_LOOP     → loop ends; proceed to post-loop deliverable.
```

## Dual-Pass Mode (Mandatory)

⛔ **Single-pass is forbidden.** All rounds MUST use Recall → fact-gather → Precision.

**⛔ Three-agent separation (inviolable)**: Recall, Fact-Gather, and Precision are THREE distinct roles that MUST NOT collapse into fewer:

| Role | Agent | Produces | Forbidden Output |
|------|-------|----------|-----------------|
| Recall | Independent subagent | Raw findings list | — |
| Fact-Gather | Main agent | Raw evidence only (quotes, line refs, structural observations) | CONFIRM/DOWNGRADE/REJECT, pre-judgments, severity suggestions |
| Precision | Independent subagent (NOT main agent) | CONFIRM/DOWNGRADE/REJECT per finding | — |

**Collapse scenarios that MUST be avoided:**
- ❌ Main agent does Fact-Gather + Precision in one step → echo chamber
- ❌ Main agent pre-judges findings during Fact-Gather → biases Precision
- ❌ Main agent filters which findings reach Precision → Recall coverage wasted
- ❌ Precision prompt contains main agent's verdicts → independence destroyed

| Phase     | Load                         | Contains                                      |
| --------- | ---------------------------- | --------------------------------------------- |
| 1–3       | `review-design.md`           | Checklist + Recall prompt + fact-gather guide |
| 4–5       | `review-code.md`             | Checklist + Recall prompt + fact-gather guide |
| Precision | `review-precision-filter.md` | Precision Filter prompt + aggregation         |

**Sequence**: Recall Pass → Gather Facts (raw only) → Precision Filter (independent subagent) → confirmed_findings → tally

Skip fact-gather → false positives pass filter → wasted fix rounds. Pre-judge during fact-gather → Precision becomes rubber stamp → dual-pass degrades to single-pass.

❌ **Reviewer prompt contamination**: injecting round counts ("Round 2 of N"), prior-round results ("R1 found X"), cumulative tallies, or fix lists into the Recall/Precision prompt. Contaminated prompts create anchoring bias — the reviewer reproduces prior conclusions instead of independent assessment.

Shared mutable state: report key-space collision/isolation and concurrency race/no-locking as separate findings. Do not treat container/key changes as race fixes. Race evidence includes unsynchronized read/write/clear, check-then-act, cache-miss loading, or clear/set interleaving; fixes require serialization, locking, single-flight/deduplication, atomic operations, transactions, or external coordination.

## Output Requirements

| Category                 | Content                                  | Required              |
| ------------------------ | ---------------------------------------- | --------------------- |
| Severity Issues          | Defects with C/H/M/P/L/I labels          | Always                |
| Constructive Suggestions | Actionable fixes paired 1:1 with C/H/M/P | When C/H/M/P exist    |
| Critical Opinions        | Strategic concerns                       | Only when substantive |

Suggestions must fix the same failure mode as their paired finding and specify what/where/why; isolation/container/rewrite fixes are not substitutes for lifecycle, capacity, or concurrency controls.

Critical Opinions must challenge reasoning, assumptions, or system direction — not restate surface symptoms.

Zero C/H/M rounds: Suggestions and Opinions optional.

## GPAV Submission

When Watchdog is active: load `ralph-gpav.md` for submission protocol.

## Stop Condition

```
gate_proceed = 2 consecutive rounds with zero new C/H/M findings
(P/L do not reset counter; known issues excluded)
```

## Next Round

After Round 1: load `ralph-continuation.md` (evaluation, flowchart, stop conditions) and `ralph-contested.md` (when C/H/M is REJECTed). Log format: `ralph-log-template.md`.
