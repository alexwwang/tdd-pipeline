## ⛔ Prerequisite: Why Articulation (MUST complete before Phase 2 execution)

Before any work in this phase, articulate your understanding of this task.
Do not proceed to execution until you have produced this reasoning.

After articulating, check: did you address what this phase protects,
where the key risks lie, and why your approach will work?
If not, supplement before proceeding.

> **Phase 2 risk hint**: Technical solution protects the traceability of architectural decisions. Skipping it means technology choices lack rationale and future change costs become unpredictable.

### ❌ What superficial Why Articulation looks like
- Listing components without explaining why this architecture addresses the identified risks
- Restating the task description ("Phase 2's goal is to design the technical solution")
- Listing steps without rationale ("I'll do A, then B")
- Dodging risk assessment ("Risks are low, just proceed normally")

---

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
2. **Boundary Review** (systematic audit per module):

Boundary check: if a requirement change triggers changes in >1 module → investigate whether the module boundary is wrong. Each module should have minimal API surface.

3. **System quality checklist**
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
    burst:          "any burst patterns beyond average load?"
    caching:        "which data is cacheable? Invalidation rules?"
  }
  maintainability: {
    change_point:   "a requirement change affects at most how many modules?"
    logic_leakage:  "is business logic centralized or scattered?"
    extension_cost: "how many modules must change to add a new capability?"
  }
}
```
4. **Classify functional points by priority**
    - Label each component, interface, and failure mode as **key** (critical path, high-risk, core business logic, or externally visible behavior) or **peripheral** (utility, helper, or low-risk)
    - This classification drives test depth in Phase 3: key functional points require comprehensive test cases (happy path, edge cases, error scenarios); peripheral items require at least basic coverage
5. **Validate priority consistency with Phase 1**
    - Forward: every Phase 1 core AC must map to ≥1 Phase 2 key component. If only peripheral components serve a core AC → reclassify a component as key OR document explicit justification
    - Reverse: key components with no core requirement trace are not errors, but document that they serve only secondary requirements (verify intentional)
    - Note: peripheral requirement → key component is acceptable (shared component serving utility + core features)
6. **Map design to Phase 1 requirements**
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

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 components, follow the Task Tree & Context Management protocol in SKILL.md.

After completing this deliverable, run the `ralph-review-loop.md` protocol (see SKILL.md §Ralph Loop Review).
Upon gate pass + user approval, proceed to Phase 3 → `phase-3-test-plan.md`.

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
  ralph:        zero C/H/M₁ issues
```

## User Approval

After the Ralph loop gate passes, present the Technical Design Document to the user for approval before proceeding to Phase 3. The user confirms:
- Architecture and component breakdown are acceptable
- Key decisions and trade-offs are understood and approved
- No fundamental concerns with the proposed approach

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1. If the user's feedback reveals a fundamental requirements flaw, return to Phase 1 (discard Phase 2 work, re-run Phase 1 Ralph loop, then restart Phase 2).


