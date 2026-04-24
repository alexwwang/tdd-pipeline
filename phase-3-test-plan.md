# Phase 3: 测试方案 (Test Plan)

## Objective

Derive a **complete test strategy** from the acceptance criteria and technical design. The test plan is the bridge between design and code.

## Detailed Process

1. **Identify core scenarios and key functional points**
    - Derive core scenarios from Phase 1 items classified as **core** (user stories and acceptance criteria)
    - Derive secondary scenarios from Phase 1 items classified as **secondary**
    - Derive key functional points from Phase 2 items classified as **key** (components, interfaces, failure modes)
    - Derive peripheral functional points from Phase 2 items classified as **peripheral**
    - List them explicitly in the "Core Scenarios & Key Functional Points" section
    - **Test depth rule**: core/key items require comprehensive test cases (happy path, edge cases, error scenarios); secondary/peripheral items require at least basic coverage (happy path + primary error path)
2. **Validate priority consistency with upstream phases**
    ```
    // Phase 1 → Phase 3 consistency
    for each core_item in Phase1.items where priority == "core":
        assert core_item appears in "Core Scenarios" section
        if core_item appears in "Secondary Scenarios":
            require explicit justification documented in "Priority Downgrade Justifications"

    // Phase 1 upgrade detection: secondary items appearing as Core Scenarios
    for each secondary_item in Phase1.items where priority == "secondary":
        if secondary_item appears in "Core Scenarios" section:
            // Possible scope creep or Phase 1 misclassification
            document in "Priority Upgrade Review" section with assessment

    // Phase 2 → Phase 3 consistency
    for each key_item in Phase2.items where priority == "key":
        assert key_item appears in "Key Functional Points" section
        if key_item appears in "Peripheral Functional Points":
            require explicit justification documented in "Priority Downgrade Justifications"

    // Phase 2 upgrade detection: peripheral items appearing as Key Functional Points
    for each peripheral_item in Phase2.items where priority == "peripheral":
        if peripheral_item appears in "Key Functional Points" section:
            // Possible scope creep or Phase 2 misclassification
            document in "Priority Upgrade Review" section with assessment

    // Cross-check: core scenario must not map entirely to peripheral functional points
    // Derivation: core_scenario → its ACs (Phase 1) → components serving those ACs
    //             (from Phase 2 "Map design to Phase 1 requirements") → those components' functional points
    for each core_scenario in "Core Scenarios":
        derived_acs       = acceptance_criteria_for(core_scenario)        // from Phase 1
        serving_components = Phase2.components_serving(derived_acs)       // from Phase 2
        derived_points     = functional_points_from(serving_components)   // from Phase 2
        if derived_points.all(priority == "peripheral"):
            // MISCLASSIFICATION: either scenario is not truly core,
            // or functional points were misclassified as peripheral
            block_until_resolved
    ```
3. **Extract and map acceptance criteria** from Phase 1 Requirements Document
    - List every criterion; none may be omitted
    - Immediately map each to tests → build the Requirements Coverage Matrix (Phase 1 → Tests)
    - Verify: every AC appears in the matrix; no AC was skipped during mapping
4. **Map each Phase 2 design element to tests**
   - Cover every component, interface, and failure mode from Phase 2
   - Build the Design Coverage Matrix (Phase 2 → Tests)
5. **Map each criterion to test type**
    - **Unit tests**: Individual functions/components in isolation (fast, deterministic)
    - **Integration tests**: Component interactions, API contracts, database calls
    - **E2E tests**: Full user flows, critical paths (minimal — only where necessary)
6. **Identify edge cases, error paths, boundary conditions**
    - Null inputs, empty collections, maximum values
    - Concurrent access, timeout scenarios, network failures
    - Invalid state transitions
7. **Define test data requirements**
    - Fixtures, mocks, factories, stubs needed
    - Shared setup vs. per-test setup
8. **Note execution-order constraints**
    - Shared fixture/setup ordering constraints (not logical test dependencies)
    - Which tests can run in parallel?
    - Reminder: no test may depend on another test passing
9. **Name every test descriptively**
    - Use `should_<expected behavior>_<context>()` naming
    - Avoid `test1()`, `test_foo()`, or implementation-centric names

## Deliverable Template

```markdown
# Test Plan: <Feature Name>

## Core Scenarios & Key Functional Points
### Core Scenarios (from Phase 1 — priority: core)
| # | Core Scenario | Source (User Story/AC) | Derived Functional Points | Test Cases |
|---|--------------|------------------------|--------------------------|------------|
| 1 | <e.g. User registration> | US-1 (Core) / AC-1 (Core) | Component: EmailValidator (Key) | happy path, invalid email, duplicate email |

### Secondary Scenarios (from Phase 1 — priority: secondary)
| # | Secondary Scenario | Source (User Story/AC) | Derived Functional Points | Test Cases |
|---|--------------------|------------------------|--------------------------|------------|
| 1 | <e.g. Profile avatar upload> | US-3 (Secondary) / AC-5 (Secondary) | Component: ImageResizer (Peripheral) | happy path, oversized file |

### Key Functional Points (from Phase 2 — priority: key)
| # | Key Functional Point | Source (Component/Interface/Failure Mode) | Test Cases |
|---|---------------------|------------------------------------------|------------|
| 1 | <e.g. Email validation service> | Component: EmailValidator (Key) | valid, invalid, empty, maxlength |
| 2 | <e.g. DB connection timeout> | Failure Mode: FM-3 (Key) | timeout handling, retry logic |

### Peripheral Functional Points (from Phase 2 — priority: peripheral)
| # | Peripheral Functional Point | Source (Component/Interface/Failure Mode) | Test Cases |
|---|----------------------------|------------------------------------------|------------|
| 1 | <e.g. Logging helper> | Component: Logger (Peripheral) | happy path |

## Requirements Coverage Matrix (Phase 1 → Tests)
// Traces Phase 1 user stories and acceptance criteria to test cases.
// For Phase 2 design element traceability, see Design Coverage Matrix below.
| # | Priority | User Story | Acceptance Criterion | Test Type | Test File | Test Name | Description |
|---|----------|-----------|---------------------|-----------|-----------|-----------|-------------|
| 1 | Core | US-1 | AC-1 | Unit | `test_x.py` | `should_create_user_with_valid_email` | Valid input creates user |
| 2 | Core | US-1 | AC-1 | Unit | `test_x.py` | `should_reject_invalid_email` | Edge case: bad format |
| 3 | Secondary | US-2 | AC-2 | Integration | `test_integration.py` | `should_send_email_on_registration` | Email service called |

## Design Coverage Matrix (Phase 2 → Tests)
| # | Priority | Design Element | Element Type | Test Type | Test File | Test Name | Description |
|---|----------|---------------|-------------|-----------|-----------|-----------|-------------|
| 1 | Key | EmailValidator | Component | Unit | `test_email.py` | `should_accept_valid_email` | Valid format |
| 2 | Key | DB timeout | Failure Mode | Integration | `test_db.py` | `should_retry_on_timeout` | Retry logic |

## Edge Cases & Error Paths
- <boundary condition>
- <error scenario>
- <concurrency scenario>

## Test Data
- <fixtures, mocks, factories needed>

## Dependencies Between Tests
- No test may depend on another test passing (TDD principle: each test is independent)
- Execution-order constraints only (e.g., shared fixture initialization, database seed setup):
  - <fixture X must be initialized before tests in `test_component_a.py` run>
  - <tests in `test_payment.py` must not run in parallel>

## Open Questions
- <environment constraint> → <resolution>
- <mock feasibility concern> → <resolution>

## Priority Downgrade Justifications
### From Phase 1 (Requirements → Test Plan)
- <item>: Core → Secondary — <justification>

### From Phase 2 (Technical Design → Test Plan)
- <item>: Key → Peripheral — <justification>

## Priority Upgrade Review
### Secondary → Core Scenarios
- <item>: Secondary → Core — <reason for upgrade or scope-creep assessment>

### Peripheral → Key Functional Points
- <item>: Peripheral → Key — <reason for upgrade or scope-creep assessment>
```

## Ralph Loop Integration

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 test groups, follow the Task Tree & Context Management protocol in SKILL.md (write index.md first, then parallel modules, then merged Ralph loop).

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Test Plan Document as the deliverable
- The Phase 1 Requirements Document and Phase 2 Technical Design Document as prior context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

## Gate: What the Reviewer Must Confirm

**Step 1 — Requirements coverage (trace to Phase 1):**
- [ ] Every user story from the Phase 1 Requirements Document maps to at least one test scenario
- [ ] Every acceptance criterion from the Phase 1 Requirements Document maps to at least one test
- [ ] Core items (labeled in Phase 1) have comprehensive test cases; secondary items have at least basic coverage

**Step 2 — Technical design coverage (trace to Phase 2):**
- [ ] Every component and interface from the Phase 2 Technical Design Document is covered by at least one test
- [ ] Every failure mode documented in Phase 2 is covered by at least one test
- [ ] Key items (labeled in Phase 2) have comprehensive test cases; peripheral items have at least basic coverage

**Step 3 — Core scenarios & key functional points completeness:**
- [ ] The plan explicitly lists core scenarios and key functional points derived from Phase 1 and Phase 2
- [ ] Each listed core scenario has complete test cases covering happy path, edge cases, and error scenarios
- [ ] Each listed key functional point has complete test cases covering happy path, edge cases, and error scenarios

**Step 4 — Cross-phase priority consistency:**
- [ ] Every core item from Phase 1 appears in the "Core Scenarios" section (not silently downgraded to secondary)
- [ ] Every key item from Phase 2 appears in the "Key Functional Points" section (not silently downgraded to peripheral)
- [ ] No secondary items from Phase 1 appear as Core Scenarios without review (potential scope creep or misclassification)
- [ ] No peripheral items from Phase 2 appear as Key Functional Points without review (potential scope creep or misclassification)
- [ ] No core scenario maps entirely to peripheral functional points (would indicate a misclassification in Phase 2 or Phase 3)
- [ ] Any priority changes (downgrades or upgrades) from upstream classifications are explicitly documented with justification

**Quality checks:**
- [ ] Edge cases and error paths are explicitly covered (core/key items: happy path + edge cases + error scenarios; secondary/peripheral items: at least happy path + primary error path)
- [ ] Test names are descriptive and behavior-focused
- [ ] Test types are appropriate (not over-relying on E2E, not under-testing integration)
- [ ] Test data requirements are explicitly specified in the deliverable (not just "feasible" — must be present)
- [ ] Test dependencies and ordering constraints are documented
- [ ] Zero C/H/M issues after Ralph loop completes

## User Approval

After the Ralph loop gate passes, present the Test Plan Document to the user for approval before proceeding to Phase 4. The user confirms:
- Test coverage is adequate for all acceptance criteria
- Edge cases and error paths are sufficiently covered
- Test strategy (unit/integration/e2e split) is appropriate

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a design flaw, return to Phase 2 (discard Phase 3 work, re-run Phase 2 Ralph loop, then restart Phase 3). If the root cause is requirements, return to Phase 1.

## Transition

Once the gate passes, proceed to **Phase 4: 测试代码**. Read `phase-4-test-code.md` for detailed instructions.
