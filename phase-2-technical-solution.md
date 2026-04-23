# Phase 2: 技术方案 (Technical Solution)

## Objective

Design the architecture and make **all major technical decisions** before writing tests or code.

## Detailed Process

1. **Planner step**: Draft high-level approach
   - Component boundaries
   - Data flow between components
   - Technology stack choices (with justification)
2. **Architect step**: Refine into concrete design
   - Module decomposition
   - Interface definitions (function signatures, API contracts, type definitions)
   - Data models and persistence strategy
3. **Critic step**: Challenge every decision
   - Find gaps, over-engineering, missing edge cases
   - Question every abstraction: is it necessary?
   - Identify failure modes the design does not handle
4. **Consensus step**: Resolve conflicts
   - Merge Planner and Architect outputs with Critic feedback
   - Produce a single, unified design document
   - Document rejected alternatives and why they were rejected
5. **Map design to Phase 1 requirements**
   - Every acceptance criterion must be achievable with this design
   - If a criterion cannot be met, flag it and propose a requirements change

## Deliverable Template

```markdown
# Technical Design: <Feature Name>

## Architecture Overview
- Diagram or text description of components
- Data flow summary

## Component Breakdown
- Component A: <responsibilities, interface, dependencies>
- Component B: <responsibilities, interface, dependencies>

## Data Models / API Contracts
- <Type definitions, function signatures, request/response schemas>

## Key Decisions
| Decision | Rationale | Alternatives Rejected |
|----------|-----------|----------------------|
| <choice> | <why> | <why not others> |

## Failure Mode Handling
- <failure scenario> → <design response>

## Open Technical Questions
- <question> → <resolution>
```

## Ralph Loop Integration

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Technical Design Document as the deliverable
- The Phase 1 Requirements Document as prior context

## Gate: What the Reviewer Must Confirm

- [ ] Design covers all acceptance criteria from Phase 1
- [ ] Interfaces are concrete enough to write tests against
- [ ] Failure modes and error paths are designed for (not just happy path)
- [ ] No over-engineering: every abstraction is justified
- [ ] Key decisions document alternatives and trade-offs
- [ ] Zero C/H/M issues after Ralph loop completes

## Transition

Once the gate passes, proceed to **Phase 3: 测试方案**. Read `phase-3-test-plan.md` for detailed instructions.
