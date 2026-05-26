---
name: double-blind-experiment-tier-2
description: |
  Standard tier of the double-blind experiment protocol. Use for internal
  adoption/rejection decisions. ≥4 scenarios, 1 scorer per scenario (N total), ≥8 GT items,
  ~9–12+N scorer sessions, 2–3 hours preparation. Conclusion: sufficient for internal
  decision-making. P-values may be cited with post-hoc caveat.
version: 3.0.0
date: 2026-05-21
---

# Double-Blind Experiment: Tier 2 — Standard

## Parameters

| Parameter | Value |
|-----------|-------|
| Scenarios | ≥ 4 (prepare 5 to allow exclusions) |
| Evaluators | 1 per variant per scenario (8 total) |
| Scorers | 1 per scenario (independent instances, no shared context) |
| GT items | ≥ 8 |
| Agent instances | ~9–12 + N scorer sessions |
| Prep time | 2–3 hours |
| Conclusion | Sufficient for internal decisions |

## Roles

| Role | Responsibilities | Must NOT |
|------|-----------------|----------|
| **Designer** | Write prompts, define variants, prepare materials | Score results |
| **Evaluator** | Receive prompts and produce reviews | Know which variant they received |
| **Scorer** | Compare outputs against ground truth, score | Know which output is variant A or B |

**No degraded configuration** — T2 requires genuinely independent scorer instances (1 per scenario, no shared context).

## Protocol Steps

### Step 1: Prepare Materials

Prepare ≥ 4 scenarios. Variant prompts and rubric are shared; targets and ground truth
are per-scenario.

```
experiment/
├── prompt-a.md
├── prompt-b.md
├── scoring-rubric.md
├── scenario-1/
│   ├── evaluator-prompt.md
│   ├── target.md
│   └── ground-truth.md
├── scenario-2/
├── scenario-3/
└── scenario-4/
```

### Step 2: Verify Delivery

```bash
# Check source prompts exist and differ
for f in experiment/prompt-a.md experiment/prompt-b.md; do
  if [ ! -f "$f" ]; then echo "MISSING: $f"; exit 1; fi
  lines=$(wc -l < "$f")
  if [ "$lines" -lt 20 ]; then echo "TOO SHORT: $f"; exit 1; fi
done
diff <(head -50 experiment/prompt-a.md) <(head -50 experiment/prompt-b.md)

# Check per-scenario files
for s in experiment/scenario-*; do
  for f in target.md ground-truth.md evaluator-prompt.md; do
    if [ ! -s "$s/$f" ]; then echo "MISSING/EMPTY: $s/$f"; exit 1; fi
  done
done
```

### Step 3: Run Evaluators

- Deliver `evaluator-prompt.md` with neutral filename
- Independent agent instances, **same model/version/provider**
- Record model identifier; prefer pinned version IDs over aliases
- If evaluator produces invalid output, re-run (≤ 3 attempts); if still invalid, exclude scenario
- **Record per-variant re-run counts**. If one variant's re-run rate exceeds the other's
  by > 2×, flag as "selection-bias risk" in the experiment log.

### Step 4: De-identify Outputs

1. Copy outputs to `response-x.md` and `response-y.md`
2. Remove variant labels and structural markers
3. Randomize X/Y assignment
4. Record mapping BEFORE scoring

**De-identification checklist**:
- [ ] Variant labels replaced with X/Y
- [ ] Structural markers normalized
- [ ] Length normalization: truncate BOTH responses to min(len_X, len_Y) × 1.5 words,
      cutting at the nearest sentence or paragraph boundary that stays within the limit.
      No marker added. Floor: never truncate below 50% of original length (if min × 1.5
      < 50% of longer response, use 50% of longer response as the limit instead).
- [ ] Metadata stripped

### Step 5: Blind Scoring (Per-Scenario Independent Sessions)

Deliver each scenario to a **separate scorer session** (fresh instance, no shared context).
For each scenario, provide: `response-x.md`, `response-y.md`, `ground-truth.md`,
`scoring-rubric.md`.

X/Y assignment is **re-randomized per scenario** (global mapping recorded in mapping.md
with per-scenario columns).

**Scorer prompt MUST include this constraint:**

> Responses with identical behavior in identical states must receive identical scores.
> Do not use structural patterns as quality proxies.

**Scorer MUST NOT receive**: variant identity, description of differences, original prompts,
or any output from other scorer sessions.

If a scorer session produces invalid output, re-run that scenario only (≤ 3 attempts).
If >1 scenario is dropped due to scorer failure, restart ALL scenarios with fresh instances.

**Attrition guard**: if scorer failures cluster on one variant's responses (e.g., the
variant produces harder-to-score outputs), flag as "selection-bias risk" in experiment log.

### Step 5b: Cross-Scenario Consistency Check

Designer identifies equivalent-state pairs BEFORE scoring (recorded in experiment plan).
Scorer never sees this classification.

After all scenarios are scored, verify scorer consistency:

1. **Check equivalent-state pairs**: for scenarios where the correct behavior is the same,
   verify that responses performing the same correct behavior received the same score
   (±10% of per-item maximum, minimum ±0.5 points).
2. **If no equivalent-state pairs exist** (all scenarios test distinct conditions):
   document this explicitly. The consistency check is vacuous and cannot provide
   evidence of scorer reliability. Note this as a limitation in the experiment report.
3. **If inconsistency found in ≥1 pair and it accounts for >20% of all comparable pairs**:
   re-score ALL scenarios with fresh scorer instances. Do not selectively re-score.

### Step 6: Interpret Results

- **≥ 4 scenarios** required. If post-exclusion count < 4, report as lower-tier conclusion.
- **Consistent direction**: ≥ 80% agreement at n ≥ 5; unanimous for n < 5
- **Magnitude screen**: weaker variant ≥ 0.9× stronger (ratio scale) or ≤ 0.3 point diff (interval)
- **P-values**: May be cited with "post-hoc" caveat unless direction was pre-registered
- Single-scenario results are anecdotal

## Ground Truth Design Rules

1. **Balanced**: ≤ 1/3 of items directly test the variable; **≥ 8 items** total
2. **Calibrated**: Mark importance based on domain judgment, not inflated
3. **Independent authorship**: Ideally written by someone who didn't design variants; at minimum,
   write GT BEFORE running evaluators

## Scoring Rubric Design

Specify: dimensions, scale (ratio or interval), aggregation method.

**Important**: If using weighted average, all dimensions must share the same scale type.
Mixing ratio and interval in a single aggregation produces meaningless threshold comparisons.

**Rubric bias check**: Verify no dimension/weight/phrasing structurally favors one variant.

## Verification Checklist

- [ ] Prompt files verified to exist, contain correct content, and differ
- [ ] Prompt delivered with neutral filename
- [ ] Evaluator instances independent (no shared context)
- [ ] Evaluator model/version/provider recorded and consistent (pinned version ID preferred)
- [ ] Evaluator outputs checked for "file not found"
- [ ] Evaluator did NOT receive ground truth
- [ ] X/Y mapping recorded before scoring
- [ ] Scorer is a different agent instance from evaluators
- [ ] Scorer received de-identified outputs only (no original prompts)
- [ ] Scorer received ground-truth.md and scoring-rubric.md
- [ ] Re-runs capped at ≤ 3 attempts per variant per scenario
- [ ] Per-variant re-run rates compared; if >2× differential, flagged as selection-bias risk
- [ ] ≥ 4 scenarios tested
- [ ] Ground truth balanced (≤ 1/3 variable-specific; ≥ 8 items)
- [ ] GT written before evaluators
- [ ] Rubric bias check performed
- [ ] Results show consistent direction (unanimous n<5; ≥80% n≥5)
- [ ] Magnitude screen passed per scenario
- [ ] If p-values cited, post-hoc caveat included (or direction pre-registered)
- [ ] Length normalization applied unconditionally to both responses in every scenario
- [ ] Each scenario scored by a separate scorer session (fresh instance, no shared context)
- [ ] X/Y assignment re-randomized per scenario (per-scenario columns in mapping.md)
- [ ] Scorer prompt included signal-pure constraint (identical behavior = identical scores)
- [ ] Cross-scenario consistency check performed (or vacuous-case documented)
- [ ] Attrition guard checked: scorer failures not clustered on one variant

## Example

```
Designer:
  1. Prepare 4 scenarios with balanced ground truth (≥ 8 items each)
  2. Write variant A and B as self-contained prompts
  3. Verify prompts exist, are non-trivial, and differ
  4. Copy assigned variants to evaluator-prompt.md
  5. Run 2 evaluators per scenario (1×A, 1×B), independent instances
  6. De-identify outputs (X/Y), record mapping
  7. Run per-scenario scorer sessions (N fresh instances) with rubric + GT + de-identified outputs
  8. Perform cross-scenario consistency check (Step 5b)
  9. Map scores back to A/B; interpret with direction, magnitude, and post-hoc caveat
```

## Notes

- Agent instances cannot run sub-instances in most tooling. Launch manually in separate
  sessions.
- LLM instances have variance — this is a feature, not a bug.
- If results contradict across scenarios, conclusion is "no significant difference."
- If one variant requires significantly more re-runs, this may indicate it is harder to
  execute or produces more edge cases — factor into interpretation.
