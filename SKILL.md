---
name: tdd-pipeline
  description: >
  A rigorous 7-phase TDD development workflow enforcing Red-Green-Refactor
  at the pipeline level: Product Design → Technical Solution → Test Plan →
  Test Code → Business Code → Pre-Release Testing → System Quality Audit.
  Phases 1–5 are creation phases with mandatory Ralph-loop review.
  Phase 6 is the pipeline's validation closure — systematic pre-release
  testing with bug root cause analysis, rollback paths, and user go/no-go
  decision. Phase 7 is a system-level quality audit with 16 bug patterns,
  integration pair discovery, and execution-order analysis — loaded on
  demand during Phase 6. No business code is written until tests exist and
  fail.
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
  - system audit
  - quality audit
  - bug pattern
  - integration gap
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

> **Core Principle**: If you can't write a failing test for it, you don't understand it.
>
> **Pace Principle**: 慢就是快，欲速不达 (Slow is fast; haste makes waste). The Ralph loop's review rounds are not overhead — they are where quality is built. Every shortcut through a gate saves minutes now but costs hours in debugging later.

## The 7 Phases

Phases 1–5 are **creation phases** — they produce artifacts reviewed by Ralph loop. Phase 6 is the **validation closure** — it validates the entire pipeline's output through systematic testing, with its own quality mechanisms (not Ralph loop). Phase 7 is a **system-level quality audit** — loaded on demand during Phase 6 when deeper pattern analysis or integration pair discovery is needed.


## Ralph Loop Review (Phases 1–5)

Phases 1–5 each end with a **mandatory Ralph-loop review** before proceeding. Phase 6 uses a different quality mechanism — see `phase-6-pre-release-testing.md` for details. See `ralph-review-loop.md` for the full protocol.

After completing each phase deliverable (Phases 1–5), run the ralph-review-loop.md protocol before user approval.



## Progressive Disclosure

**At each phase, read ONLY the corresponding `phase-N-*.md` file for detailed instructions. Do NOT load all phase files at once. The `ralph-review-loop.md` protocol is loaded automatically at each phase's review step.**

- Phase 1 → `phase-1-product-design.md` (Requirements Document)
- Phase 2 → `phase-2-technical-solution.md` (Technical Design Document)
- Phase 3 → `phase-3-test-plan.md` (Test Plan Document)
- Phase 4 → `phase-4-test-code.md` (Test Files)
- Phase 5 → `phase-5-business-code.md` (Business Code)
- Phase 6 → `phase-6-pre-release-testing.md`
  - On sub-phase failure → additionally load `phase-6-root-cause-investigation.md`
  - On pattern match or 5-Why stuck → additionally load `phase-7-system-quality-audit.md`
  - Sub-phase 1.6 (Gap Detection) → load `phase-7-system-quality-audit.md`
- Phase 7 → `phase-7-system-quality-audit.md` (system-level quality audit: 16 patterns with grep commands, pair discovery, execution order analysis)
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


