## ⛔ Prerequisite: Why Articulation (MUST complete before Phase 4 execution)

Before any work in this phase, articulate your understanding of this task.
Do not proceed to execution until you have produced this reasoning.

After articulating, check: did you address what this phase protects,
where the key risks lie, and why your approach will work?
If not, supplement before proceeding.

> **Phase 4 risk hint**: Test code protects the accuracy of tests-as-specification. Skipping it means tests may pass but verify the wrong behavior.

### ❌ What superficial Why Articulation looks like
- Writing assertions directly without explaining what invariant each assertion verifies
- Restating the task description ("Phase 4's goal is to write test code")
- Listing steps without rationale ("I'll do A, then B")
- Dodging risk assessment ("Risks are low, just proceed normally")

---

# Phase 4: 测试代码 (Test Code)

## Objective

Write **ALL tests before ANY business code**. Every test MUST fail (Red phase).

## ⚠️ TDD Enforcement Rules — Non-Negotiable

```
FORBIDDEN: [business_code, premature_implementation, passing_tests]

REQUIRE:
  tests.compile == true          # imports succeed (via minimal stubs)
  tests.run     == FAIL          # all fail at runtime (Red phase)
  test_names    =~ /^should_.*$/ # descriptive, behavior-focused
  grouping:    1 test file per logical component
  error_paths: covered           # happy path alone insufficient

stubs:
  raise NotImplementedError
  OR return values that break assertions
  # Stubs are structural placeholders, NOT business logic
```

## Deliverable Template

```markdown
# Test Execution Report (Phase 4)
- Feature: <name>
- Date: <date>
- Total tests: <N>
- Runtime failures (assertion/stub errors): <N> (expected — all should fail at runtime)
- Passed: 0 (if any pass, the test is invalid or business code leaked)
- Structural errors (compilation bugs to fix before review): 0

## Test Files
- `test_component_a.py`: <N> tests
- `test_component_b.py`: <N> tests

## Failure Summary
- `test_should_create_user`: AssertionError — stub raises NotImplementedError ✅ expected (Red phase)
- `test_should_reject_invalid_email`: AssertionError — stub returns None, expected string ✅ expected (Red phase)
```

## Ralph Loop Integration

**Before review**: Write an outline. If it contains ≥ 3 test modules or ≥ 5 test groups, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, run the `ralph-review-loop.md` protocol (see SKILL.md §Ralph Loop Review).
Upon gate pass + user approval, proceed to Phase 5 → `phase-5-business-code.md`.

**Phase 4 Code Review Specifics**: The reviewer must check:
- Are ALL tests genuinely failing? (No premature implementation?)
- Do tests import non-existent modules/functions as intended?
- Are stubs truly minimal and not passing tests?
- Is test coverage complete per the Test Plan?

## Gate: Reviewer Checklist

```
gate_pass = ALL:
  all_tests:        written per Test Plan
  compilation:      zero structural/compilation errors (imports succeed via stubs)
  runtime:          all tests fail, zero passing tests
  no_business_code: only minimal stubs that actively cause test failures
  coverage:         Test Plan fully mapped to written tests (no gaps)
  naming:           descriptive (should_<expected>_<context>), organized by component
  edge_cases:       error paths and boundary conditions covered
  ralph:            zero C/H/M issues
```

## User Approval

⛔ Prerequisite: Ralph loop has reached ✅ STOP. Do NOT present for approval before STOP.

After the Ralph loop gate passes, present the Test Execution Report to the user for approval before proceeding to Phase 5. The user confirms:
- All planned tests are written
- All tests fail as expected
- No business code exists

**If the user rejects**: Revise test files based on feedback, then re-run the Ralph loop from Round 1. If the root cause is in the test plan, return to Phase 3 (discard Phase 4 work, re-run Phase 3 Ralph loop, then restart Phase 4). If the root cause is in design or requirements, return to Phase 2 or Phase 1.
