# Ralph Loop Contested Issues

Load this file when a C/H/M issue is REJECTEDed by the main agent.
Defines the contested issue protocol, escalation rules, and invariants.

## Contested Issue Protocol

When the main agent REJECTs a C/H/M issue, it becomes a **contested issue**:

1. The main agent documents the REJECTION rationale in the review log.
2. The contested issue is included in the next round's context for the reviewer.
3. The reviewer in the next round MUST explicitly address each contested issue:
   - **Accept the rejection** (reviewer agrees the assumption was wrong)
     → issue is dropped from tally.
   - **Provide additional evidence** to re-raise the issue → issue stays in tally.
4. **Same grounds**: If the reviewer re-raises on the same grounds, the main agent
   **must** ADOPT or MODIFY. After ADOPT or MODIFY, the issue transitions from
   contested to **pending verification** and remains in the tally until the reviewer
   confirms resolution in the next round.
5. **Different grounds**: The main agent may REJECT (see §Rule 3 / Rule 4 interaction).
   If the main agent improperly REJECTs on the same grounds (violating Rule 3),
   this still counts as a dispute round toward the escalation limit.
6. **Escalation rule**: If the same issue remains contested after 2 dispute rounds,
   **escalate to the user** for resolution with a structured dossier
   (see `ralph-examples.md` Example G for the 5-section template).

A **dispute round** = one complete exchange on an issue. Round 1 = the round where
the reviewer raises the issue and the main agent REJECTs it (the issue becomes
contested at the end of this round). Each subsequent reviewer re-assessment of the
contested issue = another dispute round. Do NOT silently drop or keep contested
issues beyond 2 dispute rounds.

### Key Invariant

REJECTed C/H/M issues **remain in the gate tally** until the reviewer explicitly
drops them. The gate condition `final_round.C + .H + .M == 0` uses the tally after
contested-issue resolution, not before. Escalated issues are **suspended** from the
gate tally (pending external resolution); the gate evaluates on remaining issues only.

### Rule 3 / Rule 4 Interaction

Rule 3 (must ADOPT/MODIFY) and Rule 4 (2-round limit) are independent.

- **Same grounds with additional evidence** → Rule 3 forces ADOPT/MODIFY, blocking escalation.
- **Different grounds** → Rule 3 doesn't apply, but Rule 4 still triggers at the
  2-round limit → escalation.

## Examples

See `ralph-examples.md` for contested issue examples:

- **Example E** — contested issue lifecycle with graceful concession
- **Example F** — contested issue via MODIFY, principled compromise
- **Example G** — contested issue escalation with 5-section dossier
