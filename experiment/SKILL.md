---
name: double-blind-experiment
description: |
  Design and execute double-blind A/B experiments for AI agent workflows.
  Progressive disclosure: assess the task below, then load the appropriate
  tier file (tier-1.md, tier-2.md, or tier-3.md). Each tier file is a
  self-contained protocol for that stakes level.
author: community
version: 3.1.0
date: 2026-05-21
---

# Double-Blind Experiment Protocol

## Problem

AI agent experiments (prompt variant comparison, structural changes, skill evaluation)
are vulnerable to systematic biases that invalidate results. The most dangerous are:

1. **Delivery failure**: Prompts not actually delivered to evaluators
2. **Experimenter bias**: The designer also scores the results, knowing which is which
3. **Ground truth bias**: The person writing ground truth unconsciously favors one variant

## When to Use

- Comparing two variants of a prompt, checklist, or skill
- Testing whether any structural change improves output quality
- Evaluating any A/B change where subjective judgment is involved

Do NOT use for deterministic compliance checks (pass/fail is objectively verifiable).

## Task Assessment: Choose Your Tier

```
Is the result for an external-facing claim, release decision, or published methodology?
  YES → Load tier-3.md (Decision Grade: ≥5 scenarios, ≥2 scorers, pre-registration)
  NO → Is this for an internal adoption/rejection decision with resource at stake?
    YES → Load tier-2.md (Standard: ≥4 scenarios, 1 scorer, full checklist)
    NO → Load tier-1.md (Quick Screen: ≥3 scenarios, 1 scorer, minimal checklist)
```

## Key Definition

A **scenario** is one independent evaluation unit: a target artifact + its ground truth.
Running the same target with two agent instances produces two *replicates of one scenario*,
not two scenarios. Prepare all scenarios before starting evaluators.

## Common Failure Modes

1. **Phantom Delivery**: Evaluator says "file not found"
2. **Self-Scoring**: Orchestrator scores results themselves
3. **Single-Scenario Claims**: One data point has high variance — ⛔ 3 same-type
   scenarios pass the count check but miss defects only visible in different task types
4. **Ground Truth Favoritism**: Majority of GT items test the variable directly — ⛔
   stating the ≤ 1/3 rule does not enforce it; rubric and GT can silently violate
   without triggering any error
5. **Context Contamination**: Same agent evaluates then scores

## Pre-Experiment Design Gates

⛔ These gates MUST pass before running evaluators. If any fails, fix and re-check.

### Gate 1: Rubric Variable Balance
Variable-specific dimensions in `scoring-rubric.md` MUST be ≤ 1/3 of total.
⛔ Do not trust automated counts — if manual count disagrees, use manual count.

### Gate 2: Ground Truth Annotation Accuracy
Declared variable-specific item count in each `scenario-*/ground-truth.md` MUST
match actual count. ⛔ Do not trust AI-generated annotations — verify manually.

### Gate 3: Scenario Diversity
Scenarios MUST cover ≥ 2 distinct requirement types. Document type per scenario
in experiment plan.

## References

- Extracted from a real incident where all 5 failure modes occurred in sequence.

## Changelog

- **v3.0.0**: Progressive disclosure restructure — single entry point + self-contained tier files
- **v2.2.2**: Instance spawning guidance; verification script fixes; 17-round Ralph convergence
- **v2.0.0**: Experiment tiers (T1/T2/T3) introduced
