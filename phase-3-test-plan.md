# Phase 3: 测试方案 (Test Plan)

## Objective

Derive a **complete test strategy** from the acceptance criteria and technical design. The test plan is the bridge between design and code.

## Detailed Process

1. **Extract acceptance criteria** from Phase 1 Requirements Document
   - List every criterion; none may be omitted
2. **Map each criterion to test type**
   - **Unit tests**: Individual functions/components in isolation (fast, deterministic)
   - **Integration tests**: Component interactions, API contracts, database calls
   - **E2E tests**: Full user flows, critical paths (minimal — only where necessary)
3. **Identify edge cases, error paths, boundary conditions**
   - Null inputs, empty collections, maximum values
   - Concurrent access, timeout scenarios, network failures
   - Invalid state transitions
4. **Define test data requirements**
   - Fixtures, mocks, factories, stubs needed
   - Shared setup vs. per-test setup
5. **Note test dependencies**
   - Which tests are prerequisites for others?
   - Which tests can run in parallel?
   - Are there ordering constraints?
6. **Name every test descriptively**
   - Use `should_<expected behavior>_<context>()` naming
   - Avoid `test1()`, `test_foo()`, or implementation-centric names

## Deliverable Template

```markdown
# Test Plan: <Feature Name>

## Test Coverage Matrix
| # | Acceptance Criterion | Test Type | Test File | Test Name | Description |
|---|---------------------|-----------|-----------|-----------|-------------|
| 1 | AC-1 | Unit | `test_x.py` | `should_create_user_with_valid_email` | Valid input creates user |
| 2 | AC-1 | Unit | `test_x.py` | `should_reject_invalid_email` | Edge case: bad format |
| 3 | AC-2 | Integration | `test_integration.py` | `should_send_email_on_registration` | Email service called |

## Edge Cases & Error Paths
- <boundary condition>
- <error scenario>
- <concurrency scenario>

## Test Data
- <fixtures, mocks, factories needed>

## Dependencies Between Tests
- Test B requires Test A to exist (but not pass)
- Tests in `test_payment.py` must not run in parallel

## Open Questions
- <environment constraint> → <resolution>
- <mock feasibility concern> → <resolution>
```

## Ralph Loop Integration

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Test Plan Document as the deliverable
- The Phase 1 Requirements Document and Phase 2 Technical Design Document as prior context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

## Gate: What the Reviewer Must Confirm

- [ ] Every acceptance criterion from Phase 1 maps to at least one test
- [ ] Edge cases and error paths are explicitly covered
- [ ] Test names are descriptive and behavior-focused
- [ ] Test types are appropriate (not over-relying on E2E, not under-testing integration)
- [ ] Test data strategy is feasible
- [ ] Zero C/H/M issues after Ralph loop completes

## User Approval

After the Ralph loop gate passes, present the Test Plan Document to the user for approval before proceeding to Phase 4. The user confirms:
- Test coverage is adequate for all acceptance criteria
- Edge cases and error paths are sufficiently covered
- Test strategy (unit/integration/e2e split) is appropriate

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a design flaw, return to Phase 2 (discard Phase 3 work, re-run Phase 2 Ralph loop, then restart Phase 3). If the root cause is requirements, return to Phase 1.

## Transition

Once the gate passes, proceed to **Phase 4: 测试代码**. Read `phase-4-test-code.md` for detailed instructions.
