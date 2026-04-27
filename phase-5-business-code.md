# Phase 5: 业务代码 (Business Code)

## Objective

Implement the **minimum code** to make all tests pass, then refactor. Business code is only the implementation detail that satisfies tests.

## Detailed Process

```
phase_5():

  # Step 1: Red — already done (Phase 4)
  assert: all_tests_exist AND all_tests_fail
  FORBIDDEN: modify tests unless test proven wrong
  if test_proven_wrong:
    minor_fix (e.g. wrong expected value) → fix inline, continue
    structural_flaw → return_to_root_cause:
      tests_flawed     → Phase 4
      strategy_flawed  → Phase 3
      architecture_flawed → Phase 2
      requirements_flawed → Phase 1
      # DISCARD all work from root-cause phase onward
      # (if SPLIT=true: mark in tree.md as [DISCARDED]; if SPLIT=false: archive or revert files)
      # RE-RUN that phase's Ralph loop → re-execute subsequent phases

  # Step 2: Green
  while failing_tests.exist:
    test = pick_simplest(failing_tests)
    write(minimum_code_to_pass(test))
    run(full_test_suite)
    if test.passes AND suite.green → next failing test
    if other_tests_break → STOP (implementation too broad)
    if stuck (needs redesign):
      ESCALATE to user (propose design change OR test modification)
      FORBIDDEN: silently change test to make it easier

  RULE: no code without failing test justifying it
    want helper? → write test first
    no test justifies code? → do NOT write it

  # Step 3: Refactor
  assert: all_tests_pass before refactoring
  while smells_detected:
    refactor: [duplication, naming, logic, abstractions]
    run(full_test_suite) → MUST stay green
    log_refactoring_round()
    if tests_break:
      revert_immediately
      try_smaller_refactor (abstraction may be wrong)
  document: design_deviations_from_Phase2 with justification

  # Final Verification
  run(full_test_suite) → all_pass == true
  assert: no_business_code_without_test
  check: code_smells (duplication, long_functions, poor_names)
```

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

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 implementation files, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- All business code files, test files, and the Implementation Report as the deliverable
- The Phase 4 Test Execution Report, Phase 3 Test Plan, Phase 2 Technical Design Document, and Phase 1 Requirements Document as prior context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

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
  refactor:       complete + tests green
  lean:           minimum implementation (no over-engineering)
  deviations:     documented + justified
  ralph:          zero C/H/M issues
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

After user approval, the pipeline is complete. Invoke `tdd-pipeline` again for the next feature.
