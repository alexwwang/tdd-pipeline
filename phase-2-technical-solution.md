# Phase 2: 技术方案 (Technical Solution)

## Objective

Design the architecture and make **all major technical decisions** before writing tests or code.

## Detailed Process

Use **independent subagents** for each role (Planner, Architect, Critic) to ensure diverse perspectives.

1. **Constraints input** (before drafting):
    - Scalability envelope: expected load now vs. 12-month projection
    - Extension vectors: what new capabilities are most likely to be added?
    - Non-negotiable constraints: latency, data isolation, reversibility requirements
    - Security constraints: authentication/authorization model, encryption requirements, key management
    - Cost constraints: infrastructure budget limits, third-party service cost ceilings
    - Compliance constraints: regulatory requirements (GDPR, data residency, audit trail)
2. **Planner step**: Draft high-level approach
    - Component boundaries
    - Data flow between components
    - Technology stack choices (with justification)
3. **Architect step**: Refine into concrete design
    - Module decomposition
    - Interface definitions (function signatures, API contracts, type definitions)
    - Data models and persistence strategy
4. **Boundary Review**: Systematically audit each module boundary
    - Single responsibility check: can you describe this module's job in one sentence?
    - Dependency necessity: is each inter-module dependency justified?
    - Change impact mapping: which requirement change triggers this module to change?
    - Blast radius check: does the same requirement change also trigger other modules? (If yes → boundary may be wrong)
    - API surface check: does each module expose the minimum necessary interface? Excessive API surface is implicit coupling
5. **Security Review**: Independently audit security posture
    - Threat modeling: identify threat actors, attack surfaces, and high-risk data flows
    - Trust boundaries: map where trust level changes (external API → internal service → database)
    - Authentication & authorization: identity model, access control strategy, privilege escalation risks
    - Data protection: encryption at rest and in transit, key management, sensitive data handling
    - Input validation: where are trust boundaries? What input sanitization is needed?
    - Audit trail: which operations must be logged for compliance? What fields are required?
    - Security test scenarios: list the security test cases that Phase 3/4 must include
6. **Critic step**: Challenge every decision
    - Find gaps, over-engineering, missing edge cases
    - Question every abstraction: is it necessary?
    - Identify failure modes the design does not handle
    - **System quality checklist** (Critic must address each item):
        - Operability:
            - [ ] Concurrency/blocking: which operations must be non-blocking? Any synchronous call risks?
            - [ ] Operation reversibility: which write operations need rollback capability?
            - [ ] Resource boundaries: are there execution timeout, memory, or concurrency limits?
        - Observability:
            - [ ] Discoverability: who is notified on anomalies? Are alert conditions defined?
            - [ ] Health baseline: what are the metrics for normal operation? How is deviation defined?
            - [ ] Debuggability: are logs structured? Do critical paths have correlation IDs? Is log semantics complete?
        - Data:
            - [ ] Data isolation: which data must not enter LLM context?
            - [ ] Data loss risk: which nodes have potential data loss?
        - Security (confirm Security Review findings):
            - [ ] All threat scenarios from Security Review have a design response
            - [ ] No trust boundary gaps remain unresolved
        - Performance:
            - [ ] Latency targets: what are the response time requirements per operation?
            - [ ] Throughput: what is the expected request rate? Any burst patterns?
            - [ ] Caching strategy: which data is cacheable? Invalidation rules?
        - Maintainability:
            - [ ] Single change point: a requirement change affects at most how many modules?
            - [ ] Logic leakage: is business logic centralized or scattered across modules?
            - [ ] Extension cost: how many existing modules must change to add a new capability?
7. **Consensus step**: Resolve conflicts
    - Merge Planner and Architect outputs with Critic feedback
    - Produce a single, unified design document
    - Document rejected alternatives and why they were rejected
8. **Classify functional points by priority**
    - Label each component, interface, and failure mode as **key** (critical path, high-risk, core business logic, or externally visible behavior) or **peripheral** (utility, helper, or low-risk)
    - This classification drives test depth in Phase 3: key functional points require comprehensive test cases (happy path, edge cases, error scenarios); peripheral items require at least basic coverage
9. **Validate priority consistency with Phase 1**
    ```
    // Forward check: core requirements must be served by key components
    for each core_ac in Phase1.acceptance_criteria where priority == "core":
        parent_story = Phase1.user_story_for(core_ac)    // look up parent US regardless of its priority
        serving_components = Phase2.components_that_serve(core_ac)
        key_components     = serving_components.where(priority == "key")

        if key_components.is_empty:
            // PRIORITY INCONSISTENCY: core requirement → only peripheral components
            action = reclassify_a_component_as_key
                     OR document_explicit_justification_for_downgrade
            // e.g., "core requirement satisfied by well-tested third-party library"

    // Reverse check: orphaned key components (no core requirement trace)
    for each key_component in Phase2.items where priority == "key":
        served_requirements = Phase1.items_served_by(key_component)
        core_served         = served_requirements.where(priority == "core")

        if core_served.is_empty:
            // Not an error, but document: this key component serves only
            // secondary requirements — verify classification is intentional

    // Note: peripheral requirement → key components is acceptable
    // (utility feature may depend on critical shared component)
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

## Gate: What the Reviewer Must Confirm

- [ ] Design covers all acceptance criteria from Phase 1
- [ ] Every component, interface, and failure mode is classified as key or peripheral
- [ ] Priority consistency with Phase 1: every core requirement maps to at least one key component/interface; any downgrade is explicitly justified
- [ ] Interfaces are concrete enough to write tests against
- [ ] Failure modes and error paths are designed for (not just happy path)
- [ ] No over-engineering: every abstraction is justified
- [ ] Boundary review completed: each module passes single-responsibility, blast-radius, and API surface checks
- [ ] Security review completed: threat model, trust boundaries, data protection, and security test scenarios documented
- [ ] System quality checklist addressed: operability, observability, data, security confirmation, performance, and maintainability items confirmed
- [ ] Non-functional constraints documented in the deliverable
- [ ] Key decisions document alternatives and trade-offs
- [ ] Zero C/H/M issues after Ralph loop completes

## User Approval

After the Ralph loop gate passes, present the Technical Design Document to the user for approval before proceeding to Phase 3. The user confirms:
- Architecture and component breakdown are acceptable
- Key decisions and trade-offs are understood and approved
- No fundamental concerns with the proposed approach

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a fundamental requirements flaw, return to Phase 1 (discard Phase 2 work, re-run Phase 1 Ralph loop, then restart Phase 2).

## Transition

Once the gate passes, proceed to **Phase 3: 测试方案**. Read `phase-3-test-plan.md` for detailed instructions.
