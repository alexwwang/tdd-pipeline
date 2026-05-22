## ⛔ Prerequisite: Why Articulation (MUST complete before Phase 5 execution)

Before any work in this phase, articulate your understanding of this task.
Do not proceed to execution until you have produced this reasoning.

After articulating, check: did you address what this phase protects,
where the key risks lie, and why your approach will work?
If not, supplement before proceeding.

> **Phase 5 risk hint**: Business code protects the minimal-implementation principle. Skipping it means potential over-engineering or introducing behavior not covered by tests.

### ❌ What superficial Why Articulation looks like
- Writing code without explaining why this implementation is the minimal solution that satisfies the tests
- Restating the task description ("Phase 5's goal is to make tests pass")
- Listing steps without rationale ("I'll do A, then B")
- Dodging risk assessment ("Risks are low, just proceed normally")

---

# Phase 5: 业务代码 (Business Code)

## Objective

Implement the **minimum code** to make all tests pass, then refactor. Business code is only the implementation detail that satisfies tests.

## Deliverable Template

```markdown
# Implementation Report (Phase 5)
- Tests passing: <N>/<N>
- Lines of business code: <count>
- Refactoring rounds: <count>
- Design deviations: <any changes from Phase 2, with justification>

## File List
- `module_a.py`: <description>
- `module_b.py`: <description>
```

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 implementation files, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, run the `ralph-review-loop.md` protocol (see SKILL.md §Ralph Loop Review).
Upon gate pass + user approval, proceed to Phase 6 → `phase-6-pre-release-testing.md`.

**Phase 5 Code Review Specifics**: The reviewer must check:
- Is refactoring clean? Any regressions introduced?
- Is the minimum code principle respected? (No gold-plating?)
- Does every line of business code have test coverage?
- Are abstractions justified by the tests?
- Are there any design deviations that were not documented?

## Gate: Reviewer Checklist

```
gate_pass = ALL:
  all_tests_pass: true
  coverage:       no business code without test
  no_silent_test_modification: FORBIDDEN changing tests to make them easier to pass
  no_code_without_failing_test: every line of business code must have a failing test justifying it
  refactor:       complete + tests green
  lean:           minimum implementation (no over-engineering)
  deviations:     documented + justified
  ralph:          zero C/H/M1 issues
```

## User Approval

The user must review the final code and test suite:
- All tests pass? ✅
- No code without test coverage? ✅
- Refactoring completed? ✅

**If the user rejects**: Determine the scope of rejection:
- **Code-level issues** (quality, style, missing edge cases): Revise the code, then re-run the Phase 5 Ralph loop from Round 1.
- **Test-level issues** (wrong behavior spec, missing scenarios): Return to Phase 4, modify tests, re-run Phase 4 Ralph loop, then restart Phase 5.
- **Design or requirements issues** (wrong approach, misunderstood requirements): Return to the root-cause phase (Phase 2 or Phase 1), discard all downstream work, re-run that phase's Ralph loop, and re-execute subsequent phases.
