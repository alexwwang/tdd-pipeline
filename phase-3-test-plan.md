## ⛔ Prerequisite: Why Articulation (MUST complete before Phase 3 execution)

Before any work in this phase, articulate your understanding of this task.
Do not proceed to execution until you have produced this reasoning.

After articulating, check: did you address what this phase protects,
where the key risks lie, and why your approach will work?
If not, supplement before proceeding.

> **Phase 3 risk hint**: Test plan protects the alignment between test coverage and requirements. Skipping it means tests may miss critical scenarios or test the wrong things.

### ❌ What superficial Why Articulation looks like
- Listing test cases without explaining why these cases have their assigned priority levels (why certain scenarios are critical while others are downgraded)
- Restating the task description ("Phase 3's goal is to write the test plan")
- Listing steps without rationale ("I'll do A, then B")
- Dodging risk assessment ("Risks are low, just proceed normally")

---

# Phase 3: 测试方案 (Test Plan)

## Detailed Process

1. **Identify core scenarios and key functional points**
```
   scenarios = {
     core:      Phase1.items.where(priority == "core")       # US + AC
     secondary: Phase1.items.where(priority == "secondary")
   }
   functional_points = {
     key:       Phase2.items.where(priority == "key")         # components, interfaces, failure modes
     peripheral: Phase2.items.where(priority == "peripheral")
   }

   # Test depth rule:
   #   core/key → comprehensive (happy + edge + error)
   #   secondary/peripheral → basic (happy + primary error)
```
   List all explicitly in the "Core Scenarios & Key Functional Points" section.
2. **Validate priority consistency with upstream phases**
    ```
    # Phase 1 → Phase 3 consistency
    for each core_item in Phase1.items where priority == "core":
        assert core_item appears in "Core Scenarios" section
        if core_item appears in "Secondary Scenarios":
            require explicit justification documented in "Priority Downgrade Justifications"

    # Phase 1 upgrade detection: secondary items appearing as Core Scenarios
    for each secondary_item in Phase1.items where priority == "secondary":
        if secondary_item appears in "Core Scenarios" section:
            # Possible scope creep or Phase 1 misclassification
            document in "Priority Upgrade Review" section with assessment

    # Phase 2 → Phase 3 consistency
    for each key_item in Phase2.items where priority == "key":
        assert key_item appears in "Key Functional Points" section
        if key_item appears in "Peripheral Functional Points":
            require explicit justification documented in "Priority Downgrade Justifications"

    # Phase 2 upgrade detection: peripheral items appearing as Key Functional Points
    for each peripheral_item in Phase2.items where priority == "peripheral":
        if peripheral_item appears in "Key Functional Points" section:
            # Possible scope creep or Phase 2 misclassification
            document in "Priority Upgrade Review" section with assessment

    # Cross-check: core scenario must not map entirely to peripheral functional points
    # Derivation: core_scenario → its ACs (Phase 1) → components serving those ACs
    #             (from Phase 2 "Map design to Phase 1 requirements") → those components' functional points
    for each core_scenario in "Core Scenarios":
        derived_acs       = acceptance_criteria_for(core_scenario)        // from Phase 1
        serving_components = Phase2.components_serving(derived_acs)       // from Phase 2
        derived_points     = functional_points_from(serving_components)   // from Phase 2
        if derived_points.all(priority == "peripheral"):
            # MISCLASSIFICATION: either scenario is not truly core,
            # or functional points were misclassified as peripheral
            block_until_resolved
    ```

   edge_categories = [
     null_inputs, empty_collections, max_values,
     concurrent_access, timeouts, network_failures,
     invalid_state_transitions,
     serialization_boundary,
     error_handler_correctness,
     implicit_contract,
     resource_leak,
     cascading_failure,
     performance_logic,
   ]
```

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

_Traces Phase 1 user stories and acceptance criteria to test cases. For Phase 2 design element traceability, see Design Coverage Matrix below._
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
Checklist (verify each category is covered — find the analogous risk if category seems inapplicable):
- [ ] null_inputs — None/null passed where a value is expected
- [ ] empty_collections — empty arrays, strings, or maps fed to processing logic
- [ ] max_values — overflow, infinity, or boundary-maximum inputs
- [ ] concurrent_access — race conditions under simultaneous multi-thread/actor access (e.g., shared state if parallelized later)
- [ ] timeouts — operations that exceed time limits or receive zero/negative timeout (e.g., I/O stall on large file, infinite loop from malformed input)
- [ ] network_failures — external dependency unavailability, clock skew, connectivity loss (e.g., file I/O errors, encoding corruption from external data)
- [ ] invalid_state_transitions — operations called out of order or on corrupted state
- [ ] serialization_boundary — encoding, precision loss, schema mismatch across boundaries
- [ ] error_handler_correctness — catch blocks that swallow, amplify, or leak resources
- [ ] implicit_contract — undocumented call-order, sync/async, or thread-safety assumptions
- [ ] resource_leak — memory, FDs, connections growing over iterations (soak test)
- [ ] cascading_failure — single failure propagating through dependency chain
- [ ] performance_logic — N+1 patterns, hot paths under load, batch-vs-single inefficiency

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

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 test groups, follow the Task Tree & Context Management protocol in SKILL.md (write index.md first, then parallel modules, then merged Ralph loop).

After completing this deliverable, run the `ralph-review-loop.md` protocol (see SKILL.md §Ralph Loop Review). The reviewer should additionally load `phase-3-edge-reference.md` for edge case depth verification.
Upon gate pass + user approval, proceed to Phase 4 → `phase-4-test-code.md`.

## Gate: Reviewer Checklist

```
gate_pass = ALL:
  # Step 1 — Requirements coverage (trace to Phase 1)
  req_coverage:
    every Phase1.US → ≥ 1 test_scenario
    every Phase1.AC → ≥ 1 test
    core: comprehensive; secondary: basic_coverage

  # Step 2 — Design coverage (trace to Phase 2)
  design_coverage:
    every Phase2.component + interface → ≥ 1 test
    every Phase2.failure_mode → ≥ 1 test
    key: comprehensive; peripheral: basic_coverage

  # Step 3 — Core/key completeness
  completeness:
    core_scenarios + key_functional_points explicitly listed
    each: happy_path + edge_cases + error_scenarios

  # Step 4 — Cross-phase priority consistency
  consistency:
    Phase1.core → appears in "Core Scenarios" (not silently downgraded)
    Phase2.key → appears in "Key Functional Points" (not silently downgraded)
    no secondary→core without review (scope creep risk)
    no peripheral→key without review (scope creep risk)
    no core_scenario maps entirely to peripheral_points
    all priority_changes: documented with justification

  # Quality
  quality:
    edge_cases + error_paths explicit
    test_names: descriptive, behavior-focused
    FORBIDDEN: [test1, test_foo, impl_centric_names]
    test_types: appropriate split (not over-E2E, not under-integration)
    test_data: explicitly specified in deliverable
    test_deps + ordering: documented
    ralph: zero C/H/M issues

  # Step 5 — Advanced risk coverage (if applicable)
  advanced_coverage:
    contract_tests:    added if ≥2 deployable components with API boundary
    soak_tests:        added if long-running process or resource management
    concurrency_tests: added if shared mutable state exists
    fault_tests:       added if external dependencies or retry logic
    error_handlers:    every catch/except block has ≥1 test
    performance_tests: added if hot paths with data volume sensitivity (N+1, batch vs single)

  # Step 6 — Edge case depth verification (reviewer probes for thin coverage)
  depth_verification:
    cumulative_precision: serialization operations tested for drift over N iterations (not just single-operation accuracy)
    error_recovery: system state valid after error, retry works, resources released on failure path
    call_order_deps: operations that must complete before others verified (including across components)
    failure_isolation: non-critical dependency failure doesn't propagate to core flow
    analogous_risks: categories marked "N/A" have analogous risks identified (e.g., file I/O errors for network_failures, shared state for concurrent_access if parallelizable)
```

## User Approval

After the Ralph loop gate passes, present the Test Plan Document to the user for approval before proceeding to Phase 4. The user confirms:
- Test coverage is adequate for all acceptance criteria
- Edge cases and error paths are sufficiently covered
- Test strategy (unit/integration/e2e split) is appropriate

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a design flaw, return to Phase 2 (discard Phase 3 work, re-run Phase 2 Ralph loop, then restart Phase 3). If the root cause is requirements, return to Phase 1.
