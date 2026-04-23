# Phase 5: 业务代码 (Business Code)

## Objective

Implement the **minimum code** to make all tests pass, then refactor. Business code is only the implementation detail that satisfies tests.

## Detailed Process

### Step 1: Red (Already Done — Phase 4)
- All tests exist and fail
- Do NOT modify tests in this phase unless the test itself is proven wrong

### Step 2: Green
1. Pick the **simplest failing test**
2. Write the **minimum** code to make **only that test** pass
3. Run the **full test suite**
   - If the target test passes, move to the next failing test
   - If other tests break, stop — the implementation is too broad
4. If stuck (cannot make a test pass without major redesign):
   - Escalate to the user: propose a design change or test modification
   - Do NOT silently change the test to make it easier
5. **Rule**: No code without a failing test justifying it
   - If you want to add a helper function, write a test for it first
   - If no test justifies the code, do not write it

### Step 3: Refactor
1. **Only refactor when ALL tests pass**
2. Clean duplication, improve names, simplify logic, extract abstractions
3. Run tests **after every refactor step** — they must stay green
4. If tests break during refactor:
   - Revert immediately
   - Try a smaller, simpler refactor, or the abstraction is wrong
5. Document any design deviations from Phase 2 with justification

### Final Verification
1. Run the full test suite — all tests must pass
2. Check coverage — no business code without a test
3. Review for code smells (duplication, long functions, poor names)

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

## Ralph Loop Integration

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- All business code files, test files, and the Implementation Report as the deliverable
- The Phase 4 Test Execution Report and Phase 2 Technical Design Document as prior context

**Phase 5 Code Review Specifics**: The reviewer must check:
- Is refactoring clean? Any regressions introduced?
- Is the minimum code principle respected? (No gold-plating?)
- Does every line of business code have test coverage?
- Are abstractions justified by the tests?
- Are there any design deviations that were not documented?

## Gate: What the Reviewer Must Confirm

- [ ] All tests pass
- [ ] No code exists without test coverage
- [ ] Refactoring is complete and tests remain green
- [ ] Minimum implementation principle was followed (no over-engineering)
- [ ] Design deviations are documented and justified
- [ ] Zero C/H/M issues after Ralph loop completes

## Final Gate: User Approval

The user must review the final code and test suite:
- All tests pass? ✅
- No code without test coverage? ✅
- Refactoring completed? ✅

After user approval, the pipeline is complete. Invoke `tdd-pipeline` again for the next feature.
