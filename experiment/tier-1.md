---
name: double-blind-experiment-tier-1
description: |
  Quick Screen tier of the double-blind experiment protocol. Use for initial
  screening: is the difference large enough to warrant further testing?
  Minimal resource: ≥3 scenarios, 1 scorer, ≥6 GT items, ~7 agent instances,
  1–2 hours preparation. Conclusion strength: "promising" or "probably equivalent."
version: 3.0.0
date: 2026-05-21
---

# Double-Blind Experiment: Tier 1 — Quick Screen

## Parameters

| Parameter | Value |
|-----------|-------|
| Scenarios | ≥ 3 (prepare 4 to allow exclusions) |
| Evaluators | 1 per variant per scenario (6 total) |
| Scorers | 1 |
| GT items | ≥ 6 |
| Agent instances | ~7 |
| Prep time | 1–2 hours |
| Conclusion | "Promising" or "probably equivalent" |

## Roles

| Role | Responsibilities | Must NOT |
|------|-----------------|----------|
| **Designer** | Write prompts, define variants, prepare materials | Score results |
| **Evaluator** | Receive prompts and produce reviews | Know which variant they received |
| **Scorer** | Compare outputs against ground truth, score | Know which output is variant A or B |

**Degraded configuration allowed**: If independent agent instances are unavailable,
the Designer may act as Scorer in a separate session with: (1) a time break between
evaluation and scoring, (2) structured checklist review (not free-form), (3) explicit
acknowledgment in the experiment log that role separation was degraded.

## Protocol Steps

### Step 1: Prepare Materials

Prepare ≥ 3 scenarios. Variant prompts and rubric are shared; targets and ground truth
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
│   └── ...
└── scenario-3/
    └── ...
```

### Step 1.5: Pre-Experiment Design Gates

Run the three gates defined in `SKILL.md → Pre-Experiment Design Gates`.
No tier-specific additions at T1.

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
- Independent agent instances, same model/version/provider
- If evaluator produces invalid output, re-run (≤ 3 attempts); if still invalid, exclude scenario

### Step 4: De-identify Outputs

1. Copy outputs to `response-x.md` and `response-y.md`
2. Remove variant labels and structural markers
3. Randomize X/Y assignment
4. Record mapping BEFORE scoring

**De-identification checklist**:
- [ ] Variant labels replaced with X/Y
- [ ] Structural markers normalized
- [ ] Length tells mitigated (if outputs differ > 20%)
- [ ] Metadata stripped

### Step 5: Blind Scoring

Run 1 scorer with: `response-x.md`, `response-y.md`, `ground-truth.md`, `scoring-rubric.md`.

**Scorer MUST NOT receive**: variant identity, description of differences, original prompts.

If scorer produces invalid output, discard partial results and re-run (≤ 3 attempts).

### Step 6: Interpret Results

- **≥ 3 scenarios** required. If post-exclusion count < 3, inconclusive.
- **Consistent direction**: unanimous across scenarios (n < 5)
- **Magnitude screen**: weaker variant ≥ 0.9× stronger (ratio scale) or ≤ 0.3 point diff (interval)
- **Do not cite p-values**. Report direction and magnitude only.
- Single-scenario results are anecdotal, not evidence.

## Ground Truth Design Rules

1. **Balanced**: ≤ 1/3 of items directly test the variable; ≥ 6 items total
2. **Annotation-accurate**: Declared variable-specific item count must match actual count.
   Do not trust AI-generated annotations — verify manually. (Checked at Gate 2)
3. **Calibrated**: Mark importance based on domain judgment, not inflated for one variant
4. **Independent authorship**: Ideally written by someone who didn't design variants; at minimum,
    write GT BEFORE running evaluators

## Scoring Rubric Design

Specify: dimensions, scale (ratio or interval), aggregation method.

**Rubric bias check**: Verify no dimension/weight/phrasing structurally favors one variant.

## Verification Checklist

- [ ] Prompt files verified to exist, contain correct content, and differ
- [ ] Prompt delivered with neutral filename
- [ ] Evaluator instances independent (no shared context)
- [ ] Evaluator outputs checked for "file not found"
- [ ] Evaluator did NOT receive ground truth
- [ ] X/Y mapping recorded before scoring
- [ ] Scorer is different agent instance from evaluators (or degraded config acknowledged)
- [ ] Scorer received de-identified outputs only (no original prompts)
- [ ] Scorer received ground-truth.md and scoring-rubric.md
- [ ] Re-runs capped at ≤ 3 attempts per variant per scenario
- [ ] ≥ 3 scenarios tested
- [ ] Ground truth balanced (≤ 1/3 variable-specific; ≥ 6 items)
- [ ] GT written before evaluators
- [ ] Rubric bias check performed
- [ ] Results show consistent direction
- [ ] Magnitude screen passed per scenario

## Example

```
Designer:
  1. Prepare 3 scenarios with balanced ground truth (≥ 6 items each)
  2. Write variant A and B as self-contained prompts
  3. Verify prompts exist, are non-trivial, and differ
  4. Copy assigned variants to evaluator-prompt.md
  5. Run 2 evaluators per scenario (1×A, 1×B), independent instances
  6. De-identify outputs (X/Y), record mapping
  7. Run scorer (fresh instance) with rubric + GT + de-identified outputs
  8. Map scores back to A/B; interpret direction and magnitude only
```

## Notes

- Agent instances cannot run sub-instances in most tooling. Launch manually in separate
  sessions (browser tabs, terminals, fresh API calls).
- LLM instances have variance — same input, different instance = different output. This
  is a feature, not a bug.
- If results contradict across scenarios, conclusion is "no significant difference."
