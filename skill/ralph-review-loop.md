# Ralph Loop Review Protocol

This protocol governs **all reviews in Phases 1–5** of the TDD Pipeline. Phase 6 (Pre-Release Testing) uses a different quality mechanism — see `phase-6-pre-release-testing.md` Part 5 for the 追问 (root cause investigation) protocol, rollback paths, and user go/no-go decision.

## When to Invoke

- **Phase 6 (pre-release testing)**: Does NOT use Ralph loop. See `phase-6-pre-release-testing.md` Part 5.

## Reviewer Selection

Spawn an **independent subagent** (oracle or dedicated reviewer) that was **not involved in creating the deliverable under review**. The reviewer has no bias from the creation process, but WILL be provided with prior phase outputs for cross-phase consistency checking.

### ❌ What contaminated reviewer prompts look like

NEVER include any of the following in a reviewer's prompt — they bias the review:
| ❌ DO NOT Include | Why | ✅ Instead |
|---|---|---|
| Stop condition rules or cumulative tallies: "consecutive zero = 1, one more and we stop" | Creates confirmation bias and anchoring — reviewer looks for reasons to confirm stop | Provide ONLY the deliverable and prior phase outputs |
| Prior round findings or fix lists: "R1 found X and Y, fixed as Z" | Destroys independent judgment — reviewer evaluates against known fixes, not the deliverable itself | Present the current deliverable as-is |
| Hints about progress or remaining rounds: "this has been going on for a while" | Signals that stopping is near, discourages thorough review | Do not mention round counts or loop state |
| Narrow scope limiting: "Please verify the fix for [M-1] is correct" | Reduces full review to fix-checking — other issues may be missed | Request FULL review of the complete deliverable |
| Using generic subagent instead of dedicated reviewer | Lacks specialization; may share context with fix agent | Use independent, specialized reviewer subagent |

## Severity Classification

### ❌ What wrong severity systems look like

| ⛔ WRONG | Why |
|----------|-----|
| Fewer levels or flat list without tier structure | Loses distinction between defects and improvements, breaks stop condition counting |
| Re-label H as "accepted deviation" or "intentional simplification" | Bypasses the contested issue protocol — severity manipulation |
| Label a refactoring suggestion as M to ensure it gets fixed | Gaming the classification — M is for behavioral defects only |
| Label a real defect as P to avoid resetting the counter | Gaming the classification — conceals a defect as proposal |
Every issue MUST use exactly these 3 tiers with 6 severity levels:

### Defect Tier (缺陷层) — finite, exhaustible, counted in stop condition

Behavioral problems: the deliverable does something wrong or different from what it promises.

| Severity | Name | Definition |
|----------|------|------------|
| **C** | Critical | Fundamental flaw; deliverable is wrong, dangerous, or useless |
| **H** | High | Significant gap or serious risk |
| **M** | Major-Defect | Behavioral defect: code does something different from what its interface/documentation promises |

Classification heuristic: **"If I fix this, will the code do something differently at runtime?"** → Yes → Defect Tier.

### Quality Tier (质量层) — infinite (always improvable), NOT counted in stop condition

Organizational improvements: code works correctly but could be structured better.

| Severity | Name | Definition |
|----------|------|------------|
| **P** | Proposal | Architectural-level restructuring: extract constant, reduce coupling, consolidate logic across files |
| **L** | Low | Local improvement: rename for clarity, add comment, minor style normalization |

P vs L distinction is **scope only** (cross-file/architectural vs single-location), not nature — both are "code works but could be better."

### Observation Tier (观察层) — informational, no action required

| Severity | Name | Definition |
|----------|------|------------|
| **I** | Info | Observation, question, or suggestion with no defect |

### Cross-tier mapping (legacy compatibility)

See `severity-migration.md` for the full mapping between pre-v0.13 flat 5-level (C/H/M/L/I) and current 3-tier 6-level system.

## Review Process (Per Round)

```
for round N:
  present(deliverable, prior_phase_outputs, contested_issues_from_prior_rounds) → reviewer
  review_against(gate_criteria)              # see phase-N-*.md files
  # Cross-phase escalation (UNCONDITIONAL — cannot be overridden)
  if root_cause in prior_phase:
    HALT loop
    ESCALATE to user: "Root cause in Phase N-k. Recommend rollback to Phase N-k, discard downstream, re-run Ralph loop."
    FORBIDDEN: fix prior-phase issues in current deliverable
  # Step 1: Reviewer produces three output categories
  report:
    severity_issues: numbered with C/H/M/P/L/I labels
    constructive_suggestions: actionable fixes paired with severity issues
    critical_opinions: architectural/strategic critique (when substantive concerns exist)
  tally: C=0, H=1, M=2, P=1, L=3, I=1
  # Step 2: Main agent critical evaluation (see ralph-continuation.md)
  for each review item:
    evaluate_against(project_context) → ADOPT | REJECT | MODIFY
  # Step 3: Apply fixes
  fix: all ADOPTed and MODIFYed C + H + M (P + L + I + ADOPTed opinions optional)
  # Step 4: GPAV submission (when Watchdog is active)
  gpav_submit(round: N, tally_after_evaluation)
  log: { round: N, tally, contested, evaluation_decisions, fixes_applied, gpav_submitted: bool }
```

## Dual-Pass Review Mode

The default review mode is a **two-pass Recall/Precision pipeline**. A lighter single-pass mode is available for already-converged rounds. See dedicated files for review-specific prompts:

| Phase | Load this file | Contains |
|-------|---------------|----------|
| **1–3** (Design) | `review-design.md` | Checklist + Recall prompt + fact-gather guide |
| **4–5** (Code) | `review-code.md` | Checklist + Recall prompt + fact-gather guide |
| **Precision** (shared) | `review-precision-filter.md` | Precision Filter prompt + aggregation logic |

### When to Use

| Mode | When | Cost |
|------|------|------|
| **Dual-pass** (default) | All rounds except already-converged zero-finding rounds; complex deliverables; code review; previous round had ≥ 3 false positives | 2× LLM calls, but fewer total rounds |
| **Single-pass** | Already-converged rounds (counter ≥ 1), simple deliverables, low risk | 1× LLM call |

### Review Process (Dual-Pass Mode)

⚠️ **The three steps below MUST execute in strict sequence: Recall → fact-gather → Precision. No step may be skipped, merged, or reordered.** Precision requires raw_findings from Recall; Recall must not be contaminated by Precision concerns.

Replace Step 1 in the standard Review Process with:

```
  # Step 1a: Recall Pass
  spawn_recall_subagent(deliverable, prior_phase_outputs, contested_issues)
    → template from review-design.md (Phase 1-3) or review-code.md (Phase 4-5)
  → raw_findings
  # Step 1b: Gather facts
  facts = gather_verifiable_facts(raw_findings)
    → fact-gather guide from review-design.md or review-code.md
  # Step 1c: Precision Filter
  spawn_precision_subagent(raw_findings, facts)
    → template from review-precision-filter.md
  → confirmed_findings with CONFIRM/DOWNGRADE/REJECT verdicts
  → aggregation logic from review-precision-filter.md
  tally: from confirmed findings
  # Step 2–3: Unchanged (main agent critical evaluation + fixes)
```

## Reviewer Output Requirements

| Category | Describes | Maps to Severity Issues |
|----------|-----------|------------------------|
| **Severity-tagged Issues** | *Defects* — what is wrong | Each classified C/H/M/P/L/I |
| **Constructive Suggestions** | *Fixes* — how to resolve defects | Paired 1:1 with C/H/M/P issues; optional for L; **not required for I** |
| **Critical Opinions** | *Strategic concerns* — whether the approach is right | Independent of specific defects; optional — provide only when substantive concerns exist |

**In rounds with zero C/H/M issues**: Constructive Suggestions and Critical Opinions are both **optional**. Exempt rounds from mandatory output to avoid degenerate/performative content.

⛔ Suggestions must be directly actionable (what/where/why + concrete example when applicable). No vague advice like "consider improving X".

⛔ Critical Opinions must challenge reasoning, not surface symptoms. Provide only when genuine substantive concerns exist — do not manufacture critique to fill a template.

## GPAV Submission Protocol (Guarded Pipeline Authority Verification)

When the **Watchdog** observer is active, load `ralph-gpav.md` for the full submission protocol (format, validation rules, RPS scanner, fallback). Each review round's findings must be submitted via `ralph_round_finding` after fixes are applied — GPAV's `roundRecords` are the ground truth for stop/gate decisions.

## Gate Condition

```
gate_proceed = ALL:
  ralph_termination = stop  # 2 consecutive rounds with zero new C/H/M findings (P and L do not reset the counter)
  # escalation/rollback = model-determined, not a pass path
```

## Subsequent Round Loading

After Round 1, load `ralph-continuation.md` (evaluation, stop conditions, flowchart) and `ralph-contested.md` (only when a C/H/M issue is REJECTed). See `ralph-log-template.md` for the complete log format.
