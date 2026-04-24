# Phase 1: 产品设计 (Product Design)

## Objective

Understand **what** to build and **why**, not **how**. Surface all ambiguity before a single line of code is considered.

## Detailed Process

1. **Deep-interview the user** — Use Socratic questioning (at least 3 clarifying questions; aim for 3–5)
    - What problem does this solve?
    - Who are the actors?
    - What are the boundaries of the system?
    - What is explicitly out of scope?
2. **Identify actors, use cases, and boundaries**
    - List all user roles and system roles
    - Map primary and secondary use cases
    - Define system boundaries and external dependencies
3. **Surface ambiguity and force resolution**
    - Challenge vague terms ("fast", "secure", "user-friendly")
    - Identify unstated constraints (regulatory, performance, compatibility)
    - Do NOT proceed until every ambiguity is resolved or documented as an explicit assumption
4. **Draft user stories** in Given-When-Then format where possible
    - Format: `As a <role>, I want <goal> so that <benefit>`
    - Include informal acceptance criteria for each story (plain language)
5. **Formalize acceptance criteria** as testable Given-When-Then
    - Transform informal criteria from step 4 into explicit Given-When-Then format
    - Each criterion must be testable (binary pass/fail)
    - No criteria may use subjective language
    - Enumerate edge cases and error scenarios for each criterion
6. **Classify requirements by priority**
    - Label each user story as **core** (must-have, directly serves the primary goal) or **secondary** (supporting, nice-to-have, or low-impact)
    - Label each acceptance criterion as **core** or **secondary** following the same logic
    - This classification is consumed by Phase 2 (for architectural priority mapping) and drives test depth in Phase 3: core items require comprehensive test cases (happy path, edge cases, error scenarios); secondary items require at least basic coverage
7. **Document constraints and assumptions**
    - What are you assuming about the environment?
    - What are the known limitations?

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

## Gate: What the Reviewer Must Confirm

- [ ] All user stories are traceable to the original request
- [ ] Every acceptance criterion is testable (binary pass/fail)
- [ ] Every user story and acceptance criterion is classified as core or secondary
- [ ] Core/secondary classifications are justified: core items directly serve the primary goal; secondary items are supporting/nice-to-have
- [ ] No ambiguity remains unresolved
- [ ] Edge cases and error scenarios are identified
- [ ] Constraints and assumptions are explicit
- [ ] Zero C/H/M issues after Ralph loop completes

## User Approval

After the Ralph loop gate passes, present the Requirements Document to the user for approval before proceeding to Phase 2. The user confirms:
- All user stories accurately reflect their intent
- Acceptance criteria are complete and correct
- Constraints and assumptions are acceptable

**If the user rejects**: Revise the deliverable based on feedback, then re-run the Ralph loop from Round 1.

## Transition

Once the gate passes, proceed to **Phase 2: 技术方案**. Read `phase-2-technical-solution.md` for detailed instructions.
