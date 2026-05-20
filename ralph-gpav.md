# GPAV Submission Protocol (Guarded Pipeline Authority Verification)

Loaded when the **Watchdog** observer is active. See `ralph-review-loop.md` for the core review protocol.

## When to Submit

Submit findings via `ralph_round_finding` **after** the Main Agent Critical Evaluation (Step 2) and **after** fixes are applied (Step 3). The submission reflects the gate tally — the state that determines stop/gate decisions.

## What to Include

- **ADOPTed and MODIFYed items**: Include with their post-evaluation severity (use `original` + `downgrade_reason` when the main agent downgraded)
- **REJECTED (contested) items**: MUST be included — contested C/H/M remain in the gate tally until the reviewer explicitly drops them. Omitting them would make GPAV's tally diverge from the actual gate tally
- **L and I items**: Include as-is. For deferred (unfixed) L/I items, see `ralph-continuation.md` step 3b for documentation requirements.
- **In dual-pass mode**: Submit the Precision-filtered confirmed findings only, not the raw Recall output. The Precision Filter's CONFIRM/DOWNGRADE verdicts are the equivalent of single-pass reviewer findings

## Submission Format

```
watchdog.observe('ralph_round_finding', {
  phase: <number>,          // current TDD pipeline phase (1-5)
  round: <number>,          // current review round number
  findings: [
    {
      severity: 'C',        // C | H | M | L | I
      description: '...',   // concise issue description
    },
    {
      severity: 'M',
      description: '...',
      original: 'H',              // required when severity was downgraded
      downgrade_reason: '...',    // required and non-empty when severity < original
    },
  ]
})
```

## Validation Rules (enforced by Watchdog)

- `findings` must be an array of objects with `severity` (C/H/M/L/I) and `description` (non-empty string, max 2000 chars)
- `findings` array size limited to 50 entries per submission
- `phase` must match the current pipeline phase
- `round` must be > current `ralph.round` (incrementing — no retroactive edits)
- `downgrade_reason` must be non-empty when `original` is set and severity < original
- Duplicate submissions for the same round are rejected

## What GPAV Tracks

The Watchdog records authoritative counts per round in `roundRecords`. These counts are the **ground truth** for stop decisions — not the orchestrator's local tally. The Watchdog's `ralph_terminate` validates:

- **stop**: `roundRecords` shows ≥ 2 consecutive rounds with zero new C/H/M/L findings (findings already in the Known Issues document are excluded)
- **escalation**: model-determined — same C/H/M findings recurring despite fixes (no round cap)
- **rollback**: root cause traced to prior phase per cross-phase escalation protocol

## RPS (Review Prompt Scanner)

The Watchdog also includes a **Review Prompt Scanner** that detects prohibited content in reviewer prompts. When spawning reviewer subagents:

- **DO NOT** include review round counts, cumulative tallies, or stop-condition state in the reviewer's prompt
- **DO NOT** include prior-round fix lists or findings summaries
- **DO NOT** mention "this is round N" or "consecutive zeros = 1"

The RPS scans both `args.prompt` and `args.description` fields of `Task` tool calls during ralph_loop phase. Detection produces a WARN audit entry but does NOT block the review — it is an informational safeguard.

**In dual-pass mode**: RPS scans BOTH the Recall subagent prompt AND the Precision subagent prompt. The same prohibited-content rules apply to both — neither pass should receive round counts, tallies, or fix lists.

## GPAV Fallback (Legacy Compatibility)

When `autoValidated=true` (GPAV has been used) but `roundRecords` is empty (legacy rounds before GPAV activation), the Watchdog falls back to legacy `tallyHistory`-based validation. This path is unreachable once all rounds use `ralph_round_finding` from round 1.
