---
name: tdd-pipeline
description: >
  A rigorous 6-phase TDD development workflow enforcing Red-Green-Refactor
  at the pipeline level: Product Design → Technical Solution → Test Plan →
  Test Code → Business Code → Pre-Release Testing. Phases 1–5 are creation
  phases with mandatory Ralph-loop review. Phase 6 is the pipeline's
  validation closure — systematic pre-release testing with bug root cause
  analysis, rollback paths to earlier phases, and user go/no-go decision.
  No business code is written until tests exist and fail.
triggers:
  - tdd
  - tdd pipeline
  - test-driven
  - red green refactor
  - write tests first
  - pre-release
  - release test
  - ship it
  - go/no-go
  - ready to deploy
  - 产品设计
  - 技术方案
  - 测试驱动
  - 测试方案
  - 先写测试
  - 上线前测试
  - 发版前检查
  - 回归测试
argument-hint: <feature description or user request>
level: 3
---

# TDD Pipeline Skill

## Overview

The TDD Pipeline enforces a **strict, phase-gated workflow** where tests are the primary specification artifact. Business code is the *implementation detail* that makes tests pass — nothing more.

> **Core Principle**: If you cannot write a failing test for it, you do not understand it well enough to build it.

## The 6 Phases

Phases 1–5 are **creation phases** — they produce artifacts reviewed by Ralph loop. Phase 6 is the **validation closure** — it validates the entire pipeline's output through systematic testing, with its own quality mechanisms (not Ralph loop).

| Phase | Chinese Name | English Name | Deliverable | Quality Mechanism |
|-------|-------------|--------------|-------------|-------------------|
| 1 | 产品设计 | Product Design | Requirements Document | Ralph design review |
| 2 | 技术方案 | Technical Solution | Technical Design Document | Ralph design review |
| 3 | 测试方案 | Test Plan | Test Plan Document | Ralph design review |
| 4 | 测试代码 | Test Code | Test Files (all failing) | Ralph code review |
| 5 | 业务代码 | Business Code | Working Business Code | Ralph code review |
| 6 | 预发布测试 | Pre-Release Testing | Release Gate Checklist (with evidence) | Testing flow + 追问 + user go/no-go |

## Ralph Loop Review (Phases 1–5)

Phases 1–5 each end with a **mandatory Ralph-loop review** before proceeding. Phase 6 uses a different quality mechanism — see `phase-6-pre-release-testing.md` for details. See `ralph-review-loop.md` for the full protocol.

```
ralph_loop:
  max_rounds: 10
  reviewer: independent_subagent   # not involved in creating deliverable
  early_stop: rounds >= 2 AND consecutive_zero(round_N-1, round_N)  # zero C/H/M/L
  gate_pass: rounds >= 5 AND current_round.C+H+M == 0              # L/I acceptable
  escalation: round 10 AND C/H/M > 0 → HALT, report to user       # NOT a pass path
  termination: early_stop | gate_pass  # one must trigger to proceed
```

## Gate Rules

```
gate_pass(1–5, N → N+1) = ALL:
  deliverable(N).complete == true
  ralph_loop.termination IN [early_stop, gate_pass]   # NOT max_rounds
  final_round.severity.C + .H + .M == 0               # L/I acceptable
  TDD_INVARIANT: no business code without failing test
  user.approved == true

gate_pass(5 → 6):
  Phase 5 Ralph gate passed == true
  # Phase 6 begins automatically — no additional gate between 5 and 6

gate_pass(6 → pipeline_complete) = ALL:
  phase_6.all_sub_phases_passed == true                 # Phase 0–3 all green
  release_gate_checklist.all_checked == true            # evidence-based
  user_go_nogo_decision == GO                           # user decides, not reviewer
```

## Progressive Disclosure

**At each phase, read ONLY the corresponding `phase-N-*.md` file for detailed instructions. Do NOT load all phase files at once. The `ralph-review-loop.md` protocol is loaded automatically at each phase's review step.**

- Phase 1 → `phase-1-product-design.md`
- Phase 2 → `phase-2-technical-solution.md`
- Phase 3 → `phase-3-test-plan.md`
- Phase 4 → `phase-4-test-code.md`
- Phase 5 → `phase-5-business-code.md`
- Phase 6 → `phase-6-pre-release-testing.md`
  - On sub-phase failure → additionally load `phase-6-root-cause-investigation.md`
- Task tree → `task-tree.md` (loaded ONLY when Split Decision evaluates to SPLIT=true)

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Fix |
|-------------|----------------|-----|
| Business code before tests | Violates TDD, creates untested code | STOP → write test first |
| Test passes immediately | Business code leaked or test is wrong | Remove leaked code or fix test |
| Change tests to fit impl | Tests are the spec, not the code | Fix the code, not the test |
| Skip refactor step | Accumulates technical debt | Always refactor when green |
| Bypass Ralph gates (Phase 1–5) | Hidden flaws propagate downstream | Run until gate pass, enforce zero M+ |
| One giant test file | Poor organization, hard to maintain | 1 file per component/module |
| Only happy-path tests | Misses real-world failures | MUST test errors + boundaries |
| Phase 6 partial re-run | Fixes may introduce regressions | Always full re-run from Phase 6 Phase 0 |
| Skip 追问 when Phase 6 fails | Fixes symptom, not root cause | Run Layer Isolation + 5-Why + T1-T3 |

## Split Decision

Before writing any phase deliverable, evaluate structural complexity:

```
write_outline()                    // list planned sections, modules, stories, components

# "modules" and "stories" are phase-adaptive:
#   Phase 1: stories = user stories
#   Phase 2: stories = components
#   Phase 3: stories = test groups
#   Phase 4: stories = test modules
#   Phase 5: stories = implementation files
if count(modules) >= 3 OR count(stories) >= 5:
    SPLIT = true
    if count(modules) < 3:
        # Stories triggered the split; group stories into 3-7 cohesive modules
        modules = group_by_cohesion(stories, target=clamp(count(stories)/2, 3, 7))
    elif count(modules) > 7:
        # Too many modules; merge small ones into larger cohesive groups
        modules = merge_smallest(modules, target=7)
    # Result: 3-7 modules, each a cohesive group for parallel execution
else:
    SPLIT = false                                    // write as single document
```

If `SPLIT = true`, load **`task-tree.md`** for the decomposition, execution, context-reading, and versioning protocols. If `SPLIT = false`, proceed normally with a single deliverable document.

## Rollback from Phase 6

When Phase 6 discovers issues, the 追问 (root cause investigation) determines which phase to roll back to:

| Root Cause Layer | Rollback Target | Rerun Scope |
|-----------------|-----------------|-------------|
| Test gap (missing coverage) | Phase 4 | Rerun Phase 4 → 5 → 6 |
| Code implementation bug | Phase 5 | Rerun Phase 5 → 6 |
| Design/architecture flaw | Phase 2 | Rerun Phase 2 → 3 → 4 → 5 → 6 |
| Requirement misunderstanding | Phase 1 | Rerun full pipeline |
| Config/environment only | Fix config | Rerun Phase 6 only (full rerun) |

See `phase-6-pre-release-testing.md` for the complete 追问 protocol with termination criteria and rollback procedures.

## Exit Conditions

```
pipeline_complete = ALL:
  phases[1..6].gate_passed == true
  all_tests.pass == true
  no_business_code_without_test_coverage == true
  release_gate_checklist.all_checked == true
  user.approved == true

# Approval granularity (set at invocation):
#   approval_mode=every_phase  (default) — approve at each phase boundary
#   approval_mode=final_only   — approve only at Phase 6 end
```
