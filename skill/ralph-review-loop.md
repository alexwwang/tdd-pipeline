# Ralph Loop Review Protocol

Phases 1–5 only. Phase 6 uses `phase-6-pre-release-testing.md` Part 5.

## Reviewer Selection

Spawn an **independent subagent** not involved in creating the deliverable. Provide prior phase outputs for cross-phase consistency.

**Reviewer prompt MUST NOT contain**: stop conditions, cumulative tallies, prior round findings, fix lists, round counts, loop state, or scope-limiting hints.

## Severity Classification

Three tiers with six levels:

### Defect Tier (counted in stop condition)

| Sev | Name | Definition |
|-----|------|------------|
| C | Critical | Fundamental flaw; deliverable is wrong, dangerous, or useless |
| H | High | Significant gap or serious risk |
| M | Major | Behavioral defect: code does something different from interface/documentation promise |

**Heuristic**: "If I fix this, will runtime behavior change?" → Yes → Defect Tier.

### Quality Tier (NOT counted)

| Sev | Name | Definition |
|-----|------|------------|
| P | Proposal | Cross-file/architectural restructuring |
| L | Low | Single-location improvement (rename, comment, style) |

### Observation Tier

| Sev | Name | Definition |
|-----|------|------------|
| I | Info | Observation or question with no defect |

Legacy mapping: `severity-migration.md`

## Review Process

```
for round N:
  deliverable + prior_phase_outputs + contested_issues → reviewer
  reviewer → severity_issues (C/H/M/P/L/I) + constructive_suggestions + critical_opinions
  main_agent evaluates each item → ADOPT | REJECT | MODIFY
  fix all ADOPTed/MODIFYed C/H/M (P/L/I/ADOPTed_opinions optional)
  log: { round, tally, contested, evaluation_decisions, fixes_applied }
```

## Dual-Pass Mode (Mandatory)

⛔ **Single-pass is forbidden.** All rounds MUST use Recall → fact-gather → Precision.

| Phase | Load | Contains |
|-------|------|----------|
| 1–3 | `review-design.md` | Checklist + Recall prompt + fact-gather guide |
| 4–5 | `review-code.md` | Checklist + Recall prompt + fact-gather guide |
| Precision | `review-precision-filter.md` | Precision Filter prompt + aggregation |

**Sequence**: Recall Pass → Gather Facts → Precision Filter → confirmed_findings → tally

Skip fact-gather → false positives pass filter → wasted fix rounds.

## Output Requirements

| Category | Content | Required |
|----------|---------|----------|
| Severity Issues | Defects with C/H/M/P/L/I labels | Always |
| Constructive Suggestions | Actionable fixes paired 1:1 with C/H/M/P | When C/H/M/P exist |
| Critical Opinions | Strategic concerns | Only when substantive |

Zero C/H/M rounds: Suggestions and Opinions optional.

## GPAV Submission

When Watchdog is active: load `ralph-gpav.md` for submission protocol.

## Stop Condition

```
gate_proceed = 2 consecutive rounds with zero new C/H/M findings
```

P and L do not reset the counter.

## Next Round

After Round 1: load `ralph-continuation.md` (evaluation, flowchart, stop conditions) and `ralph-contested.md` (when C/H/M is REJECTed). Log format: `ralph-log-template.md`.
