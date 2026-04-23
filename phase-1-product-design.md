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
   - Include acceptance criteria for each story
5. **Define acceptance criteria explicitly**
   - Each criterion must be testable (binary pass/fail)
   - No criteria may use subjective language
6. **Document constraints and assumptions**
   - What are you assuming about the environment?
   - What are the known limitations?

## Deliverable Template

```markdown
# Requirements Document: <Feature Name>

## User Stories
- As a <role>, I want <goal> so that <benefit>

## Acceptance Criteria
1. Given <context>, When <action>, Then <expected result>
2. ...

## Constraints & Assumptions
- <explicit assumptions made>
- <known limitations>

## Open Questions (must be resolved before Phase 2)
- <question 1> → <resolution>
```

## Ralph Loop Integration

After completing this deliverable, **invoke `ralph-review-loop.md`** with:
- The Requirements Document as the deliverable
- The user's original request as context

**Cross-phase escalation**: If the reviewer identifies a root cause in a prior phase during the Ralph loop, follow the cross-phase escalation protocol in `ralph-review-loop.md` step 3 (halt loop, recommend rollback to user).

## Gate: What the Reviewer Must Confirm

- [ ] All user stories are traceable to the original request
- [ ] Every acceptance criterion is testable (binary pass/fail)
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
