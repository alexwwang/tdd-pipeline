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

## Detailed Process

```
phase_4():

  # 1. Create test files — one per component, mirroring planned business code
  for component in test_plan:
    create_test_file(component)

  # 2. Write test cases importing non-existent business code
  for test_case in test_plan:
    import(module_from_non_existent_code)  # intentional: defines public interface

  # 3. Create minimal stubs (ALWAYS required — imports must succeed)
  for module in imported_modules:
    create_stub(module):
      raise NotImplementedError
      # OR return dummy value guaranteed to break assertions

  # 4. Run tests and confirm ALL FAIL
  run(all_tests)
  for failure in test_results:
    record(failure_mode) → Test Execution Report :: Failure Summary  # roadmap for Phase 5
  if any_test_passes:
    investigate("business code leaked OR test is wrong")

  # 5. Map failures to Test Plan
  for planned_test in test_plan:
    assert: planned_test in written_tests
    if cannot_write:
      escalate → Phase 1 or Phase 3
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

> **Note**: All imports must succeed (via minimal stubs). If any import fails, create the missing stub before proceeding to review. Import errors are NOT acceptable at gate time — only runtime assertion failures are expected.

## Ralph Loop Integration

**Before review**: Write an outline. If it contains ≥ 3 test modules or ≥ 5 test groups, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- All test files and the Test Execution Report as the deliverable
- The Phase 3 Test Plan, Phase 2 Technical Design, and Phase 1 Requirements Document as prior context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

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

After the Ralph loop gate passes, present the Test Execution Report to the user for approval before proceeding to Phase 5. The user confirms:
- All planned tests are written
- All tests fail as expected
- No business code exists

**If the user rejects**: Revise test files based on feedback, then re-run the Ralph loop from Round 1. If the root cause is in the test plan, return to Phase 3 (discard Phase 4 work, re-run Phase 3 Ralph loop, then restart Phase 4). If the root cause is in design or requirements, return to Phase 2 or Phase 1.

## Transition

Once the gate passes, proceed to **Phase 5: 业务代码**. Read `phase-5-business-code.md` for detailed instructions.
