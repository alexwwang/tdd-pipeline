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

  # ── STEP 0: Workload Assessment ─────────────────────────────────────────────
  # Before dispatching any subagent, the main agent estimates workload:
  #
  #   estimated_context = prompt_tokens + deliverable_size
  #   if estimated_context > 40% of subagent max_context → SPLIT_REQUIRED
  #   if findings_count > 30 → SPLIT_REQUIRED (Recall/Precision only)
  #
  # If SPLIT_REQUIRED:
  #   Create a delegation plan (see §Workload Split Protocol below).
  #   Dispatch sub-tasks sequentially or in parallel per plan.
  #   Save each sub-task output to a temp file.
  #   Merge outputs before proceeding to next step.
  #
  # Temp file naming: r{round}-{stage}-{sub_task_id}.json
  #   e.g. r1-recall-mod-auth.json, r1-factgather-cluster-001.json

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

  # ── STEP B: Fact-Gathering (independent subagent) ────────────────────────────
  # Load review-fact-gather.md. Inject RAW_FINDINGS + phase-specific guide.
  # ⛔ Fact-Gather subagent MUST be independent from Recall and Precision.
  # ⛔ Fact-Gather subagent produces ONLY location references (paths + scopes).
  #    It MUST NOT contain: evaluation, quotes, summaries, or any judgment.
  # Dispatch: recall_output + {FACT_INVESTIGATION_GUIDE} → Fact-Gather subagent
  # Output: location_map = { <finding_id>: [ {path, scope, scope_description} ] }
  # ⛔ Do NOT dispatch Precision subagent until location_map is non-empty.
  location_map ← Fact-Gather subagent(recall_output)

  # ⛔ MANDATORY OUTPUT — emit the following block verbatim before proceeding to Step C.
  # Step C MUST NOT begin until this block appears in the output.
  #
  #   [FACT-GATHER COMPLETE]
  #   findings_received: <N from Recall output>
  #   locations_mapped: <M entries in location_map>
  #   location_map = {
  #     <finding_id>: [{path, scope, scope_description}]
  #     ...
  #   }
  #   blocker: NONE | <reason if locations_mapped = 0>
  #
  # Validity rules (self-check before continuing):
  #   - findings_received > 0 and locations_mapped > 0  → proceed to Step C
  #   - findings_received > 0 and locations_mapped = 0  → emit blocker, HALT
  #   - findings_received = 0                           → emit blocker, HALT
  #
  # ⛔ locations_mapped = 0 while findings_received > 0 is a protocol violation.
  # ⛔ Emitting evaluation or quoted content in location_map is a protocol violation.

  # ── STEP C: Precision Filter ──────────────────────────────────────────────────
  # Load review-precision-filter.md. Inject location_map + recall_output.
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
  #   2. location_map (document locations to investigate — NOT quoted content)
  #   3. The Precision filter instructions (from review-precision-filter.md)
  # The Precision prompt MUST NOT contain:
  #   1. Main agent's pre-judgment on any finding
  #   2. "Suggested" or "recommended" verdicts
  #   3. Hints about which findings are "probably false positives"
  #
  # WHY: Main agent has context bias (it built the recall prompt, dispatched
  # fact-gather, and formed opinions). Independent Precision breaks this bias loop.
  # If main agent = Precision agent, dual-pass collapses to echo chamber.
  confirmed_findings ← Precision subagent(location_map, recall_output)

  # ── STEP D: Evaluate + Fix + Scan + Log ───────────────────────────────────────
  # The Eval-Fix subagent evaluates confirmed findings and produces fix suggestions.
  # The main agent then mechanically applies fixes and performs cross-reference scan.

  # D1: Eval-Fix subagent evaluates each confirmed finding
  # Load review-evalfix.md. Inject confirmed findings + deliverable + context.
  # ⛔ Same prompt contamination rules as Step A apply.
  # ⛔ Eval-Fix subagent MUST NOT directly edit files — output suggestions only.
  # Output: ADOPT | REJECT | MODIFY | DEFER per finding, with fix suggestions.
  reviewer_decisions ← Eval-Fix subagent(confirmed_findings)

  # D2: Apply fixes (main agent — mechanical execution, no judgment)
  for each ADOPT/MODIFY in reviewer_decisions:
      apply_fix(decision.fix_suggestion, decision.fix_code)

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

⛔ **Single-pass is forbidden.** All rounds MUST use Recall → fact-gather → Precision → Reviewer.

**⛔ Five-agent separation (inviolable)**: Recall, Fact-Gather, Precision, Reviewer, and Main Agent are FIVE distinct roles that MUST NOT collapse into fewer:

| Role | Agent | Produces | Forbidden Output |
|------|-------|----------|-----------------|
| Recall | Independent subagent | Raw findings list | — |
| Fact-Gather | Independent subagent | Document location index (paths + scopes) | Evaluation, quotes, summaries, any judgment |
| Precision | Independent subagent (NOT main agent) | CONFIRM/DOWNGRADE/REJECT per finding | — |
| Eval-Fix | Independent subagent (NOT main agent) | ADOPT/REJECT/MODIFY/DEFER + fix suggestions | Direct file edits |
| Main Agent | Orchestrator | Dispatch, merge, apply fixes, log | Any review judgment or evaluation |

**Collapse scenarios that MUST be avoided:**
- ❌ Fact-Gather + Precision merged → Fact-Gather's location role polluted by judgment
- ❌ Precision + Reviewer merged → confirm/reject judgment biases fix suggestions
- ❌ Main agent executes any review pass → context bias destroys independence
- ❌ Fact-Gather outputs quoted content or opinions → biases Precision's independent reading
- ❌ Reviewer directly edits files → bypasses cross-reference scan

| Phase     | Load                              | Contains                                      |
| --------- | --------------------------------- | --------------------------------------------- |
| 1–3       | `review-design.md`                | Checklist + Recall prompt + investigation guide |
| 4–5       | `review-code.md`                  | Checklist + Recall prompt + investigation guide |
| Fact-Gather | `review-fact-gather.md`         | Location-mapping prompt + anti-corruption rules |
| Precision | `review-precision-filter.md`      | Precision Filter prompt + aggregation         |
| Eval-Fix | `review-evalfix.md`              | Evaluation prompt + fix suggestion template    |

**Sequence**: Recall Pass → Locate Documents (Fact-Gather) → Precision Filter (independent) → Reviewer (evaluate + suggest fixes) → Main Agent (apply + scan + log)

Skip fact-gather → Precision lacks location context → relies solely on LLM judgment. Fact-Gather outputs opinions → biases Precision → dual-pass degrades to single-pass.

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

## Workload Split Protocol

Before dispatching any subagent, the main agent estimates whether the task fits within
a single subagent's context window.

### Assessment

```
workload_estimate(prompt_tokens, deliverable_size, findings_count):
    estimated_context = prompt_tokens + deliverable_size

    if estimated_context > 40% of subagent max_context:
        return SPLIT_REQUIRED

    # For Recall/Precision, also check findings count
    if findings_count > 30:
        return SPLIT_REQUIRED

    return SINGLE_DISPATCH
```

**40% threshold**: The prompt must leave ≥60% of context for the subagent to read
files, reason, and produce output.

### Split Strategies

When `SPLIT_REQUIRED`, split by the following dimension depending on the stage:

| Stage | Split Dimension | Method |
|-------|----------------|--------|
| Recall | Deliverable modules | Partition by module boundary from `task-tree.md` index.md |
| Fact-Gather | Findings clusters | Cluster findings by relevant file, dispatch per cluster |
| Precision | Same clusters as Fact-Gather | Keep findings and locations paired |
| Reviewer | Fix file overlap | Group confirmed findings by file overlap |

### Delegation Plan

When splitting, the main agent creates a delegation plan:

1. Log: `[WORKLOAD SPLIT] {stage} split into {K} sub-tasks`
2. Save plan to: `{yymmdd-summary}/split-plan-p{N}-r{round}.md`
3. Dispatch sub-tasks (parallel where independent, sequential where dependent)
4. Save each sub-task output: `r{round}-{stage}-{sub_task_id}.json`
5. Merge outputs before proceeding to next pipeline step

## Task Naming Convention

All files and tasks use the following naming convention:

- **Requirement ID**: `yymmdd-summary` format (e.g. `260608-user-auth`)
- **TDD phase identifier**: `p1`–`p8` (not the phase name)
- **Review stage identifier**: `recall`, `factgather`, `precision`, `evalfix`

**File naming patterns**:

| Type | Pattern |
|------|---------|
| Deliverable | `{yymmdd-summary}/p{N}-{desc}.md` |
| Review temp | `r{round}-{stage}-{sub_task_id}.json` |
| Review log | `{yymmdd-summary}/review-log-p{N}-r{round}.md` |
| Split plan | `{yymmdd-summary}/split-plan-p{N}-r{round}.md` |
