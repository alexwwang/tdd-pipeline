# Phase 1: 产品设计 (Product Design)

## Objective

Understand **what** to build and **why**, not **how**. Surface all ambiguity before a single line of code is considered.

## Detailed Process

```
phase_1():
  # Step 1: Deep-interview
  ask_clarifying_questions(min=3, target=3..5):
    - "What problem does this solve?"
    - "Who are the actors?"
    - "What are the system boundaries?"
    - "What is explicitly out of scope?"

  # Step 2: Identify scope
  identify: [user_roles, system_roles, primary_use_cases, secondary_use_cases, system_boundaries, external_deps]

  # Step 3: Surface ambiguity — FORCE resolution
  for vague_term in ["fast", "secure", "user-friendly", ...]:
    challenge(vague_term) → concrete_definition | explicit_assumption
  for unstated in [regulatory, performance, compatibility]:
    surface(unstated) → document
  HALT_UNTIL: all ambiguities resolved OR documented as assumptions

  # Step 4: Draft user stories
  format: "As a <role>, I want <goal> so that <benefit>"
  include: informal_acceptance_criteria (plain language per story)

  # Step 5: Formalize acceptance criteria
  transform: informal_criteria → Given-When-Then format
  constraints:
    each_criterion.testable == true        # binary pass/fail
    no_subjective_language == true
    enumerate: [edge_cases, error_scenarios] per criterion

  # Step 6: Classify by priority
  for item in [user_stories, acceptance_criteria]:
    label(item) ∈ {core, secondary}
    core:     "must-have, directly serves primary goal"
    secondary: "supporting, nice-to-have, low-impact"
  # → consumed by Phase 2 (architectural priority mapping)
  # → drives Phase 3 test depth:
  #     core → comprehensive (happy + edge + error)
  #     secondary → basic (happy + primary error)

  # Step 7: Document constraints & assumptions
  document: [environment_assumptions, known_limitations]
```

## Deliverable Template

```markdown
# Requirements Document: <Feature Name>

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

## Ralph Loop Integration

**Before review**: Write an outline. If it contains ≥ 3 modules or ≥ 5 user stories, follow the Task Tree & Context Management protocol in SKILL.md (index.md first → parallel modules → merged Ralph loop).

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Requirements Document as the deliverable
- The user's original request as context
- Use the **Review Log Template** in `ralph-review-loop.md` to record all rounds

**Cross-phase escalation**: Not applicable to Phase 1 (no prior phases exist). If the reviewer identifies a fundamental flaw in the original user request, escalate directly to the user.

## Gate: Reviewer Checklist

```
gate_pass = ALL:
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

## Transition

Once the gate passes, proceed to **Phase 2: 技术方案**. Read `phase-2-technical-solution.md` for detailed instructions.
