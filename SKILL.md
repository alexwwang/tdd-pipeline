---
name: tdd-pipeline
description: >
  A rigorous 5-phase TDD development workflow enforcing Red-Green-Refactor
  at the pipeline level: Product Design → Technical Solution → Test Plan →
  Test Code → Business Code. Each phase produces a deliverable and requires
  Ralph-loop review with zero M+ issues before proceeding. No business code
  is written until tests exist and fail.
triggers:
  - tdd
  - tdd pipeline
  - test-driven
  - red green refactor
  - write tests first
  - 产品设计
  - 技术方案
  - 测试驱动
  - 测试方案
  - 先写测试
argument-hint: <feature description or user request>
level: 3
---

# TDD Pipeline Skill

## Overview

The TDD Pipeline enforces a **strict, phase-gated workflow** where tests are the primary specification artifact. Business code is the *implementation detail* that makes tests pass — nothing more.

> **Core Principle**: If you cannot write a failing test for it, you do not understand it well enough to build it.

## The 5 Phases

| Phase | Chinese Name | English Name | Deliverable | Review Type |
|-------|-------------|--------------|-------------|-------------|
| 1 | 产品设计 | Product Design | Requirements Document | Ralph design review |
| 2 | 技术方案 | Technical Solution | Technical Design Document | Ralph design review |
| 3 | 测试方案 | Test Plan | Test Plan Document | Ralph design review |
| 4 | 测试代码 | Test Code | Test Files (all failing) | Ralph code review |
| 5 | 业务代码 | Business Code | Working Business Code | Ralph code review |

## Ralph Loop Review

Every phase ends with a **mandatory Ralph-loop review** before proceeding. See `ralph-review-loop.md` for the full protocol.

- Up to **10 review rounds** total
- Early stop: allowed at any round ≥ 2 when the previous 2 rounds were both zero-issue (zero C/H/M/L — this is stricter than gate pass, which tolerates L issues)
- Floor: if early stop never triggers, run at least 5 rounds before evaluating gate pass
- If C/H/M persist after 10 rounds, halt and escalate to user
- Full protocol with decision flowchart and examples → see `ralph-review-loop.md`
- Gate: **zero C/H/M issues** remaining (L/I are acceptable)
- Spawn an **independent reviewer subagent** for each round

## Gate Rules

To proceed from phase N to phase N+1, ALL of the following must be true:
1. Phase N deliverable is complete
2. Ralph loop terminated via early stop (round ≥ 2 consecutive zero) OR gate pass (round ≥ 5, zero C/H/M); if max rounds reached (10) without resolution, escalate to user — do NOT proceed
3. Final round has **zero M+ (C/H/M) issues** — L and I are acceptable
4. **TDD is non-negotiable**: No business code until tests exist and fail
5. **User has approved** the deliverable before proceeding

## Progressive Disclosure

**At each phase, read ONLY the corresponding `phase-N-*.md` file for detailed instructions. Do NOT load all phase files at once. The `ralph-review-loop.md` protocol is loaded automatically at each phase's review step.**

- Phase 1 → `phase-1-product-design.md`
- Phase 2 → `phase-2-technical-solution.md`
- Phase 3 → `phase-3-test-plan.md`
- Phase 4 → `phase-4-test-code.md`
- Phase 5 → `phase-5-business-code.md`
- Task tree → `task-tree.md` (loaded ONLY when Split Decision evaluates to SPLIT=true)

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | What To Do Instead |
|-------------|----------------|-------------------|
| Writing business code before tests | Violates TDD, creates untested code | Stop. Write the test first. |
| Writing tests that pass immediately | Business code leaked or test is wrong | Remove code, or fix the test |
| Changing tests to fit implementation | Tests are the spec, not the code | Fix the code, not the test |
| Skipping the refactor step | Accumulates technical debt | Always refactor when green |
| Bypassing Ralph loop gates | Hidden flaws propagate downstream | Run rounds until early stop or 10 max, enforce zero M+ |
| Writing one giant test file | Poor organization, hard to maintain | One test file per component/module |
| Only testing happy paths | Misses real-world failures | Explicitly test errors and boundaries |

## Split Decision

Before writing any phase deliverable, evaluate structural complexity:

```
write_outline()                    // list planned sections, modules, stories, components

// "modules" and "stories" are phase-adaptive:
//   Phase 1: stories = user stories
//   Phase 2: stories = components
//   Phase 3: stories = test groups
//   Phase 4: stories = test modules
//   Phase 5: stories = implementation files
if count(modules) >= 3 OR count(stories) >= 5:
    SPLIT = true
    if count(modules) < 3:
        // Stories triggered the split; group stories into 3-7 cohesive modules
        modules = group_by_cohesion(stories, target=clamp(count(stories)/2, 3, 7))
    elif count(modules) > 7:
        // Too many modules; merge small ones into larger cohesive groups
        modules = merge_smallest(modules, target=7)
    // Result: 3-7 modules, each a cohesive group for parallel execution
else:
    SPLIT = false                                    // write as single document
```

If `SPLIT = true`, load **`task-tree.md`** for the decomposition, execution, context-reading, and versioning protocols. If `SPLIT = false`, proceed normally with a single deliverable document.

## Exit Conditions

The pipeline is complete when:
1. All 5 phases finished and Ralph loops passed
2. All tests pass
3. No business code without test coverage
4. User has approved the final result. Approval granularity is set at pipeline invocation:
   - `approval_mode=every_phase` (default): user approves at each phase boundary
   - `approval_mode=final_only`: user approves only at Phase 5 end
