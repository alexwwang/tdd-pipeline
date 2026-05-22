# Ralph Loop — Review Log Template

Loaded once at the start of each Ralph loop. See `ralph-review-loop.md` for the core protocol.

```
## Ralph Loop Review Log: Phase <N> — <Phase Name>

### Round 1 [dual-pass]
- Review mode: dual-pass (Recall → Precision)
- Recall Pass: {N} raw findings
- Precision Pass: {M} confirmed, {K} rejected
- C: 0 | H: 1 | M: 1 | P: 1 | L: 2 | I: 1
- Issues (after precision filter):
  - [H-1] ...  (Recall F-03 → CONFIRM)
  - [M-1] ...  (Recall F-07 → DOWNGRADE from H)
  - [M-2] ...  (Recall F-01 → CONFIRM)
- Rejected by Precision Filter:
  - Recall F-12 → REJECT: score() uses .get(), no KeyError possible
  - Recall F-17 → REJECT: no self._baseline reference in codebase
- Constructive Suggestions:
  - <paired with C/H/M/P issues; optional for L; not required for I>
- Critical Opinions (if any):
  - <substantive critique, or omit if no genuine concerns>
- Main Agent Critical Evaluation:

  | Item | Decision | Rationale | Action |
  |------|----------|-----------|--------|
  | [H-1] | ADOPT | ... | Fixed: ... |
  | [M-1] | MODIFY | ... | Adapted: ... |
  | [M-2] | REJECT | ... | No change (contested — see next round) |

- Fixes applied: ...
- GPAV submitted: yes/no (round N → {C:0, H:1, M:1, P:1, L:2, I:1})
- Contested issues forwarded to next round:

  | Issue | First contested | Dispute round | Reviewer action needed |
  |-------|----------------|---------------|----------------------|
  | [M-2] | Round 1 | 1 of 2 | Accept rejection OR re-raise with evidence |

### Round 2
- Contested from prior round:
  - [M-2] (disputed Round 1): Reviewer response → <ACCEPT rejection / RE-RAISE with evidence: ...>
  - Resolution: <dropped from tally / remains in tally>
- C: 0 | H: 0 | M: 0 | P: 1 | L: 1 | I: 0
- Issues:
  - [M-1] ...
- Constructive Suggestions:
  - ...
- Critical Opinions:
  - (omitted — no substantive concerns this round)
- Main Agent Critical Evaluation:

  | Item | Decision | Rationale | Action |
  |------|----------|-----------|--------|
  | [M-1] | ADOPT | ... | Fixed: ... |

- Fixes applied: ...
- GPAV submitted: yes (round N → {C:0, H:0, M:0, P:1, L:1, I:0})
- Contested issues forwarded to next round: (none)

... (continue until consecutive-zero counter reaches 2) ...

### Final Gate
- Final round C/H/M count (after contested resolution): 0
- Proceed to Phase <N+1>: YES / NO
```
