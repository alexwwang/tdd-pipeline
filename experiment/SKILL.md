---
name: double-blind-experiment
description: |
  Design and execute double-blind A/B experiments for AI agent workflows.
  Progressive disclosure: assess the task below, then load the appropriate
  tier file (tier-1.md, tier-2.md, or tier-3.md). Each tier file is a
  self-contained protocol for that stakes level.
author: community
version: 3.0.0
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

### Execution-stage (caught by protocol)

1. **Phantom Delivery**: Evaluator says "file not found" — verify files exist before running
2. **Self-Scoring**: Orchestrator scores results themselves — delegate to independent agent
5. **Context Contamination**: Same agent evaluates then scores — scorer must be fresh instance

### Design-stage (caught by pre-experiment gates)

3. **Single-Scenario Claims**: One data point has high variance — run ≥ 3 scenarios
   - Root cause: Protocol requires ≥N scenarios but does not enforce diversity.
     Three scenarios of the same type pass the count check but miss defects
     only visible in a different task type.
   - Fix: Gate 3 requires ≥ 2 distinct requirement types.

4. **Ground Truth Favoritism**: Majority of GT items test the variable directly — cap at ≤ 1/3
   - Root cause: Protocol states the ≤ 1/3 rule but has no enforcement mechanism.
     Rubric and GT can violate the rule without triggering any error.
   - Fix: Gate 1 auto-counts variable-specific dimensions; Gate 2 verifies
     declared annotation matches actual count.

## References

- Extracted from a real incident where all 5 failure modes occurred in sequence.

## Changelog

- **v3.0.0**: Progressive disclosure restructure — single entry point + self-contained tier files
- **v2.2.2**: Instance spawning guidance; verification script fixes; 17-round Ralph convergence
- **v2.0.0**: Experiment tiers (T1/T2/T3) introduced
