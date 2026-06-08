# Precision Filter Prompt (Shared)

This file is loaded when dual-pass review mode is active. Used by both design review
(Phase 1–3) and code review (Phase 4–5).

## Precision Filter Prompt Template

Inject this prompt into the Precision subagent. Placeholders `{LOCATION_MAP}`,
`{RAW_FINDINGS}`, and `{REVIEW_SCOPE}` are filled by the main agent after the Recall
pass and Fact-Gather step.

```text
⛔ If {LOCATION_MAP} is empty or missing, HALT and request locations from the main
   agent. Do NOT proceed with judgment based solely on LLM reasoning.

You are a senior review quality auditor (Precision Filter Pass).
Another reviewer has completed an initial scan and found {N} potential issues.
A document locator has identified the relevant code sections for each finding.
Your job: read those sections, verify the validity and severity of each finding,
and filter out false positives.

## Review Scope (forwarded from Recall — do not modify)

{REVIEW_SCOPE}

⛔ Findings that are OUT OF SCOPE (Recall tagged [DEFERRED] or severity I with an
   out-of-scope reason): verdict MUST be REJECT, reason = "OUT OF SCOPE: belongs to
   phase {target phase}". Do NOT CONFIRM or UPGRADE these to C/H/M regardless of
   how strong the evidence appears.

## Document Locations

The following document locations were identified by the Fact-Gather subagent as relevant
to each finding. You MUST read the actual content at these locations before making a
verdict — do NOT rely on the location descriptions alone.

{LOCATION_MAP}

## Findings to Verify

{RAW_FINDINGS}

## Output Format

For each finding:
[F-XX] VERDICT: CONFIRM | DOWNGRADE | REJECT
Adjusted Severity: (only for CONFIRM/DOWNGRADE)
Reason: {one-sentence justification}

## Verification Criteria

1. Factual accuracy: Does the issue actually exist? (verify against facts, not
   speculation)
2. Scope compliance: Is the finding within IN SCOPE as declared in {REVIEW_SCOPE}?
   Not in scope → force REJECT (see above). Apply before all other criteria.
3. Severity appropriate: Is the severity classification accurate? (neither inflated
   nor deflated; applies to IN SCOPE findings only)
4. Evidence sufficient: Does the evidence support the description?
5. Actionability: Can the suggestion be directly executed?
6. Root cause dedup: Multiple surface symptoms of the same root cause should be
   merged into one finding.

## Known False Positive Patterns (watch for these)

- Reviewer misread code intent (e.g. flagging an intentional architectural decision
  as a defect)
- Reviewer lacked project context (e.g. flagging a framework convention as an
  anti-pattern)
- Reviewer over-generalized (e.g. flagging a locally reasonable pattern as a global
  problem)
- Reviewer reported multiple surface symptoms of the same root cause separately
- Reviewer speculated without evidence (e.g. claiming a property doesn't exist
  without verifying)
- Reviewer raised issues outside the current phase scope (e.g. Phase 3/5 concerns
  in a Phase 2 review) → force REJECT as scope violation, not a false positive
```

## Aggregation Logic

After Precision Pass, map confirmed findings into the Ralph tally:

```text
for finding in precision_results:
    if finding.verdict == "CONFIRM":
        tally[finding.adjusted_severity] += 1
    elif finding.verdict == "DOWNGRADE":
        tally[finding.adjusted_severity] += 1  # downgraded but retained
    elif finding.verdict == "REJECT":
        pass  # filtered out

# Confidence-based auto-downgrade for confirmed findings:
# confidence < 0.5 → auto-downgrade one severity level
# (C→H, H→M, M→L, L→I)
```

## Cost-Benefit Reference

Based on empirical testing:

| Metric                     | Value                                      |
| -------------------------- | ------------------------------------------ |
| Raw findings per round     | 25 → 12 (after Precision filter)           |
| True H-level findings      | 2 (found with higher confidence)           |
| H-level false positives    | 0                                          |
| Effective finding rate     | ~92%                                       |
| Expected Ralph loop rounds | 3–5 (vs 5–7 without Precision filter)      |

⛔ **Single-pass is forbidden.** The value is not cost reduction but quality
improvement — preventing H-level false positives from triggering wasted fix rounds.
One wasted round costs the same as an entire dual-pass round.
