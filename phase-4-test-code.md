# Phase 4: 测试代码 (Test Code)

## Objective

Write **ALL tests before ANY business code**. Every test MUST fail (Red phase).

## ⚠️ TDD Enforcement Rules — Non-Negotiable

1. **No business code yet** — only test files
2. **Every test must compile/import** but **fail when run**
3. **Tests define the interface** — they are the contract
4. **Use descriptive test names** — `should_reject_negative_amount()` not `test1()`
5. **Group by behavior** — one test file per logical component
6. **Include error path tests** — happy path alone is insufficient
7. **No premature implementation** — stubs must throw or return dummy values so tests still fail

## Detailed Process

1. **Create test files** based on the Test Plan
   - One file per component/module
   - Mirror the structure of the planned business code
2. **Write test cases** importing the (non-existent) business code
   - Import functions, classes, and modules that do not exist yet
   - This is intentional — it defines the public interface
3. **Create minimal stubs ONLY if required for compilation**
   - Empty functions, class skeletons, or type definitions
   - Stubs must `raise NotImplementedError` or return clearly wrong values
   - **Stubs are NOT business logic** — they are structural placeholders
4. **Run tests and confirm they ALL FAIL**
   - Document the failure modes — they are the roadmap for Phase 5
   - If any test passes, investigate immediately: business code leaked, or the test is wrong
5. **Map failures to the Test Plan**
   - Ensure every planned test is represented
   - Note any tests that could not be written (escalate to Phase 1 or 3)

## Deliverable Template

```markdown
# Test Execution Report (Phase 4)
- Total tests: <N>
- Failed: <N> (expected — all should fail)
- Passed: 0 (if any pass, the test is invalid or business code leaked)
- Errors: <compilation/import errors to fix>

## Test Files
- `test_component_a.py`: <N> tests
- `test_component_b.py`: <N> tests

## Failure Summary
- `test_should_create_user`: ImportError — `create_user` does not exist ✅ expected
- `test_should_reject_invalid_email`: AssertionError — stub raises wrong exception ✅ expected
```

## Ralph Loop Integration

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

## Gate: What the Reviewer Must Confirm

- [ ] All tests are written per the Test Plan
- [ ] All tests fail as expected (zero passing tests)
- [ ] No business logic has been written (only minimal stubs allowed)
- [ ] Stubs do not accidentally make tests pass
- [ ] Tests are descriptively named and organized by component
- [ ] Error paths and edge cases are covered
- [ ] Zero C/H/M issues after Ralph loop completes

## User Approval

After the Ralph loop gate passes, present the Test Execution Report to the user for approval before proceeding to Phase 5. The user confirms:
- All planned tests are written
- All tests fail as expected
- No business code exists

**If the user rejects**: Revise test files based on feedback, then re-run the Ralph loop from Round 1. If the user identifies issues rooted in the test plan, return to Phase 3 (discard Phase 4 work, re-run Phase 3 Ralph loop, then restart Phase 4). If the root cause is design or requirements, return to Phase 2 or Phase 1.

## Transition

Once the gate passes, proceed to **Phase 5: 业务代码**. Read `phase-5-business-code.md` for detailed instructions.
