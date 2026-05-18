## ⛔ Prerequisite: Why Articulation (MUST complete before Phase 1 execution)

Before any work in this phase, articulate your understanding of this task.
Do not proceed to execution until you have produced this reasoning.

After articulating, check: did you address what this phase protects,
where the key risks lie, and why your approach will work?
If not, supplement before proceeding.

> **Phase 1 risk hint**: Product design protects the accuracy of requirements understanding. Skipping it means all subsequent technical decisions rest on wrong premises.

### ❌ What superficial Why Articulation looks like
- Writing user stories directly without explaining why these stories cover the core requirements
- Restating the task description ("Phase 1's goal is to complete product requirements")
- Listing steps without rationale ("I'll do A, then B")
- Dodging risk assessment ("Risks are low, just proceed normally")
- Accepting vague terms like "fast" or "user-friendly" without challenging them to concrete definitions

---

# Phase 1: 产品设计 (Product Design)

## Objective

Understand **what** to build and **why**, not **how**. Surface all ambiguity before a single line of code is considered.

## Deliverable Template

```markdown
# Requirements Document: <Feature Name>

## System Boundaries
- **In scope**: <what this feature must do>
- **Out of scope**: <what is explicitly excluded>
- **External dependencies**: <services, APIs, libraries this feature depends on>

## User Stories
| # | Priority | User Story |
|---|----------|-----------|
| US-1 | Core | As a <role>, I want <goal> so that <benefit> |
| US-2 | Secondary | As a <role>, I want <goal> so that <benefit> |

## Acceptance Criteria
| # | User Story | Priority | Acceptance Criterion | Edge Cases |
|---|-----------|----------|---------------------|------------|
| AC-1 | US-1 | Core | Given <context>, When <action>, Then <expected result> | <edge case 1, edge case 2> |
| AC-2 | US-2 | Secondary | Given <context>, When <action>, Then <expected result> | <edge case 1> |

## Constraints & Assumptions
- <explicit assumptions made>
- <known limitations>

## Open Questions (must be resolved before Phase 2)
- <question 1> → <resolution>
```

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 user stories, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

## Gate: Reviewer Checklist

```
gate_pass = ALL:
  boundaries:     system scope, exclusions, and external deps explicitly defined
  traceability:   all user_stories → traceable to original request
  testability:    every AC testable (binary pass/fail, no subjective language)
  classification: every US + AC ∈ {core, secondary}
  justification:  core/secondary labels justified per definition
  ambiguity:      zero unresolved ambiguities
  edge_cases:     error scenarios + boundary conditions identified
  constraints:    assumptions + limitations explicit
  ralph:          zero C/H/M issues
```

## User Approval

After the Ralph loop gate passes, present the Requirements Document to the user for approval before proceeding to Phase 2. The user confirms:
- All user stories accurately reflect their intent
- Acceptance criteria are complete and correct
- Constraints and assumptions are acceptable

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1.


