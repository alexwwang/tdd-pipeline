# Phase 2: 技术方案 (Technical Solution)

## Objective

Design the architecture and make **all major technical decisions** before writing tests or code.

## Detailed Process

Use **independent subagents** for each role (Planner, Architect, Critic) to ensure diverse perspectives.

1. **Constraints input** (before drafting):
```
REQUIRED_DIMENSIONS = {
  scalability:    { current: <load>, projection_12mo: <load> }
  extension:      <likely new capabilities>
  non_negotiable: [latency, data_isolation, reversibility]
  security:       { auth_model, encryption, key_management }
  cost:           { infra_budget, third_party_ceiling }
  compliance:     [GDPR, data_residency, audit_trail]
}
```
2. **Planner step**: Draft high-level approach
    - Component boundaries
    - Data flow between components
    - Technology stack choices (with justification)
3. **Architect step**: Refine into concrete design
    - Module decomposition
    - Interface definitions (function signatures, API contracts, type definitions)
    - Data models and persistence strategy
4. **Boundary Review** (systematic audit per module):
```
for module in modules:
  assert: one_sentence_job(module)            # can you describe this module's job in one sentence?
  assert: all_deps_justified(module)           # is each inter-module dependency justified?
  for req_change in requirement_changes:
    triggered = modules_affected_by(req_change)  # which requirement change triggers this module?
    if len(triggered) > 1 → investigate boundary  # may indicate wrong split
  assert: min_api_surface(module)              # excessive API surface is implicit coupling
```
5. **Security Review** (independent audit):
```
security_review = {
  threat_model:    [actors, attack_surfaces, high_risk_data_flows]
  trust_boundaries: map(trust_level_changes)  # e.g. external API → internal → DB
  auth:            { identity_model, access_control, privilege_escalation_risks }
  data_protection: { encryption: [at_rest, in_transit], key_mgmt, sensitive_data }
  input_validation: { trust_boundaries, sanitization_points }
  audit_trail:     { required_logged_ops, required_fields }
  test_scenarios:  → Phase 3/4 security test cases
}
```
6. **Critic step**: Challenge every decision
    - Find gaps, over-engineering, missing edge cases
    - Question every abstraction: is it necessary?
    - Identify failure modes the design does not handle
    - **System quality checklist** (Critic must address each):
```
quality = {
  operability: {
    concurrency:    "which operations must be non-blocking? Any sync call risks?"
    reversibility:  "which write operations need rollback capability?"
    resources:      "execution timeout, memory, or concurrency limits?"
  }
  observability: {
    alerts:         "who is notified on anomalies? Are alert conditions defined?"
    health:         "what are normal-operation metrics? How is deviation defined?"
    debug:          "are logs structured? Do critical paths have correlation IDs?"
  }
  data: {
    isolation:      "which data must not enter LLM context?"
    loss_risk:      "which nodes have potential data loss?"
  }
  security: {  # confirm Security Review findings
    all_threats_addressed:  "all threat scenarios have a design response?"
    trust_boundary_gaps:    "no trust boundary gaps remain unresolved?"
  }
  performance: {
    latency:        "response time requirements per operation?"
    throughput:     "expected request rate? Any burst patterns?"
    caching:        "which data is cacheable? Invalidation rules?"
  }
  maintainability: {
    change_point:   "a requirement change affects at most how many modules?"
    logic_leakage:  "is business logic centralized or scattered?"
    extension_cost: "how many modules must change to add a new capability?"
  }
}
```
7. **Consensus step**: Resolve conflicts
    - Merge Planner and Architect outputs with Critic feedback
    - Produce a single, unified design document
    - Document rejected alternatives and why they were rejected
8. **Classify functional points by priority**
    - Label each component, interface, and failure mode as **key** (critical path, high-risk, core business logic, or externally visible behavior) or **peripheral** (utility, helper, or low-risk)
    - This classification drives test depth in Phase 3: key functional points require comprehensive test cases (happy path, edge cases, error scenarios); peripheral items require at least basic coverage
9. **Validate priority consistency with Phase 1**
    ```
    # Forward check: core requirements must be served by key components
    for each core_ac in Phase1.acceptance_criteria where priority == "core":
        parent_story = Phase1.user_story_for(core_ac)    // look up parent US regardless of its priority
        serving_components = Phase2.components_that_serve(core_ac)
        key_components     = serving_components.where(priority == "key")

        if key_components.is_empty:
            # PRIORITY INCONSISTENCY: core requirement → only peripheral components
            action = reclassify_a_component_as_key
                     OR document_explicit_justification_for_downgrade
            # e.g., "core requirement satisfied by well-tested third-party library"

    # Reverse check: orphaned key components (no core requirement trace)
    for each key_component in Phase2.items where priority == "key":
        served_requirements = Phase1.items_served_by(key_component)
        core_served         = served_requirements.where(priority == "core")

        if core_served.is_empty:
            # Not an error, but document: this key component serves only
            # secondary requirements — verify classification is intentional

    # Note: peripheral requirement → key components is acceptable
    # (utility feature may depend on critical shared component)
    ```
10. **Map design to Phase 1 requirements**
    - Every acceptance criterion must be achievable with this design
    - If a criterion cannot be met, flag it and propose a requirements change

## Deliverable Template

```markdown
# Technical Design: <Feature Name>

## Architecture Overview
- Diagram or text description of components
- Data flow summary

## Component Breakdown
| Component | Priority | Responsibilities | Serves Phase 1 ACs | Interface | Dependencies |
|-----------|----------|-----------------|---------------------|-----------|-------------|
| Component A | Key | <responsibilities> | AC-1, AC-2 | <interface> | <dependencies> |
| Component B | Peripheral | <responsibilities> | AC-3 | <interface> | <dependencies> |

## Data Models / API Contracts
- <Type definitions, function signatures, request/response schemas>

## Key Decisions
| Decision | Rationale | Alternatives Rejected |
|----------|-----------|----------------------|
| <choice> | <why> | <why not others> |

## Failure Mode Handling
| Failure Scenario | Priority | Design Response |
|-----------------|----------|----------------|
| <failure scenario> | Key | <design response> |
| <failure scenario> | Peripheral | <design response> |

## Non-functional Constraints
| Dimension | Requirement | Design Response |
|-----------|-------------|-----------------|
| Concurrency/blocking | <requirement> | <response> |
| Operation reversibility | <requirement> | <response> |
| Data isolation | <requirement> | <response> |
| Resource boundaries | <requirement> | <response> |
| Extension vectors | <requirement> | <response> |
| Authentication/authorization | <requirement> | <response> |
| Encryption | <requirement> | <response> |
| Latency targets | <requirement> | <response> |
| Throughput | <requirement> | <response> |
| Cost constraints | <requirement> | <response> |
| Compliance | <requirement> | <response> |

## Observability Design
| Signal | Metric / Log | Alert Condition | Owner |
|--------|-------------|-----------------|-------|
| <health indicator> | <how measured> | <threshold> | <who> |

## Cost Estimation
| Item | Type | Estimated Cost | Notes |
|------|------|---------------|-------|
| <infrastructure/component> | One-time / Recurring | <$amount> | <justification> |
| <third-party service> | Recurring | <$amount/month> | <usage basis> |
| <development overhead> | One-time | <hours or $> | <which modules> |

## Priority Downgrade Justifications
- <downgraded item>: <original priority in Phase 1> → <new priority in this design> — <justification>

## Open Technical Questions
- <question> → <resolution>
```

## Ralph Loop Integration

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 components, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Technical Design Document as the deliverable
- The Phase 1 Requirements Document as prior context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

## Gate: Reviewer Checklist

```
gate_pass = ALL:
  coverage:     all Phase1.AC covered by design
  classification: all components/interfaces/failure_modes ∈ {key, peripheral}
  consistency:  Phase1.core → maps_to ≥ 1 Phase2.key  # downgrades justified
  testability:  interfaces concrete enough for test authoring
  failure:      error paths designed (not just happy path)
  lean:         every abstraction justified (no over-engineering)
  boundary:     single_responsibility + blast_radius + min_api_surface ✓
  security:     threat_model + trust_boundaries + data_protection + test_scenarios ✓
  quality:      operability + observability + data + performance + maintainability ✓
  nfr:          non-functional constraints documented
  decisions:    alternatives + trade-offs recorded
  ralph:        zero C/H/M issues
```

## User Approval

After the Ralph loop gate passes, present the Technical Design Document to the user for approval before proceeding to Phase 3. The user confirms:
- Architecture and component breakdown are acceptable
- Key decisions and trade-offs are understood and approved
- No fundamental concerns with the proposed approach

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a fundamental requirements flaw, return to Phase 1 (discard Phase 2 work, re-run Phase 1 Ralph loop, then restart Phase 2).

## Transition

Once the gate passes, proceed to **Phase 3: 测试方案**. Read `phase-3-test-plan.md` for detailed instructions.
