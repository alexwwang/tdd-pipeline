# Ralph Loop Contested Issues

Load this file when either condition triggers:
- A C/H/M issue is REJECTed by the main agent, OR
- The main agent **downgrades** the same C/H/M issue for the 2nd time across rounds (see §Downgrade Trigger)

Defines the contested issue protocol, escalation rules, and invariants.

## Contested Issue Protocol

### Contested State Block (Mandatory Output)

After every REJECT or DOWNGRADE decision on a C/H/M issue, the main agent MUST emit
the following block before performing any fix work or logging for that round.
One block per affected issue_id.

```
[CONTESTED STATE: <issue_id>]
current_decision:          REJECT | DOWNGRADE | ADOPT | MODIFY
prior_downgrade_count:     <N>   ← increments each time this issue was previously downgraded/rejected
dispute_rounds:            <N>   ← 0 on first rejection; increments each round reviewer re-raises
grounds_same_as_prior:     YES | NO | N/A (N/A on first rejection)
escalation_due:            NO | YES
next_action:               ADOPT | MODIFY | REJECT | ESCALATE_TO_USER
```

Validity rules (self-check before emitting next_action):
- `current_decision = REJECT`  and  `grounds_same_as_prior = YES`  →  next_action MUST be ADOPT or MODIFY  (Rule 3)
- `dispute_rounds >= 2`                                             →  escalation_due = YES  →  next_action MUST be ESCALATE_TO_USER  (Rule 4)
- `escalation_due = YES`  and  next_action ≠ ESCALATE_TO_USER      →  protocol violation
- `next_action = REJECT`  while  `dispute_rounds >= 2`             →  protocol violation

⛔ Omitting this block for any REJECT or DOWNGRADE decision is a protocol violation.
⛔ Writing next_action = REJECT after dispute_rounds >= 2 is a protocol violation.
⛔ Writing next_action = REJECT when grounds_same_as_prior = YES is a protocol violation.
⛔ Fix work or logging MUST NOT begin until this block is emitted and next_action is valid.

---

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

### Downgrade Trigger (2nd downgrade of same issue)

A **downgrade** = the main agent relabels a C/H/M finding as P/L/I, or MODIFYs it
to a lower severity tier (C→H→M→P→L→I). Downgrading is a legitimate evaluation
decision in isolation, but **repeated downgrade of the same issue** signals a
disagreement that should enter the contested protocol.

**Trigger condition**: When the reviewer raises an issue as C/H/M and the main
agent has previously downgraded or rejected the **same substantive issue** in a
prior round, the 2nd occurrence triggers the contested issue protocol:

- Round N: reviewer finds [M-3] Missing timeout handling → main agent relabels as P
  → **Record as first downgrade** in review log (not yet contested)
- Round N+1: reviewer re-raises the same issue as [M-3] → main agent downgrades again
  → **Contested issue triggered** → load this file → apply full contested protocol

**Matching rule**: "Same substantive issue" = same location AND same underlying
problem (wording may differ). The main agent must track downgraded C/H/M items
across rounds to detect re-raises.

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
