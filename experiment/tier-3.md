---
name: double-blind-experiment-tier-3
description: |
  Decision Grade tier of the double-blind experiment protocol. Use for
  external-facing claims, release decisions, or published methodology.
  Maximum rigor: ≥5 scenarios, ≥2 scorers, ≥10 GT items, pre-registration,
  independent GT/rubric review, secret salt for X/Y assignment. ~15+
  agent instances, 4+ hours preparation. Conclusion defensible externally.
version: 3.0.0
date: 2026-05-21
---

# Double-Blind Experiment: Tier 3 — Decision Grade

## Parameters

| Parameter | Value |
|-----------|-------|
| Scenarios | ≥ 5 (prepare 6 to allow exclusions) |
| Evaluators | 1 per variant per scenario (10+ total) |
| Scorers | ≥ 2, report inter-scorer agreement |
| GT items | ≥ 10 (ideally 12–15) |
| Agent instances | ~15+ |
| Prep time | 4+ hours |
| Conclusion | Defensible for external claims |

## Roles

| Role | Responsibilities | Must NOT |
|------|-----------------|----------|
| **Designer** | Write prompts, define variants, prepare materials | Score results |
| **Evaluator** | Receive prompts and produce reviews | Know which variant they received |
| **Scorer** (≥ 2) | Compare outputs against ground truth, score | Know which output is variant A or B |

**No degraded configuration** — T3 requires genuinely independent scorer instances.

## Protocol Steps

### Step 1: Prepare Materials

Prepare ≥ 5 scenarios. Variant prompts and rubric are shared; targets and ground truth
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
├── scenario-4/
└── scenario-5/
```

**Pre-registration** (mandatory): Record predicted winning variant and justification in
experiment log BEFORE running any evaluator. This record is required for p-value interpretation.

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
- Record model identifier; **pinned version IDs required** (not aliases)
- If provider does not expose version hashes, note this as a known risk
- If evaluator produces invalid output, re-run (≤ 3 attempts); if still invalid, exclude scenario
- **Record per-variant re-run counts**. If one variant's re-run rate exceeds the other's
  by > 2×, flag as "selection-bias risk" in the experiment log.

### Step 4: De-identify Outputs

1. Copy outputs to `response-x.md` and `response-y.md`
2. Remove variant labels and structural markers
3. **Use a deterministic scheme with secret salt** unknown to the scorer to determine
   X/Y assignment. Publish the scheme and salt only AFTER scoring is complete.
4. Record mapping BEFORE scoring

**De-identification checklist**:
- [ ] Variant labels replaced with X/Y
- [ ] Structural markers normalized
- [ ] Length tells mitigated (if outputs differ > 20%)
- [ ] Metadata stripped
- [ ] X/Y assignment uses deterministic scheme with secret salt
- [ ] Scheme and salt published only after scoring

### Step 5: Blind Scoring

Run **≥ 2 independent scorers** with identical inputs: `response-x.md`, `response-y.md`,
`ground-truth.md`, `scoring-rubric.md`.

**Scorer MUST NOT receive**: variant identity, description of differences, original prompts.

If scorer produces invalid output, discard partial results and re-run (≤ 3 attempts).

**Multi-scorer protocol**: Report inter-scorer agreement per scenario (direction match).
If scorers disagree on direction for any scenario, flag for adjudication. Adjudication:
a third independent scorer (or Designer, blind to variant identity) reviews both scored
outputs and determines direction. Record outcome. If adjudication is infeasible, exclude
the disputed scenario from direction count. Overall conclusion requires majority agreement
across all non-excluded scenarios.

### Step 6: Interpret Results

- **≥ 5 scenarios** required. If post-exclusion count < 5, downgrade conclusion to matching tier.
- **Consistent direction**: ≥ 80% agreement at n ≥ 5; unanimous for n < 5
- **Magnitude screen**: weaker variant ≥ 0.9× stronger (ratio scale) or ≤ 0.3 point diff (interval)
- **P-values**: May be cited without post-hoc caveat if direction was pre-registered
- Single-scenario results are anecdotal

## Ground Truth Design Rules

1. **Balanced**: ≤ 1/3 of items directly test the variable; **≥ 10 items** total (ideally 12–15)
2. **Calibrated**: Mark importance based on domain judgment, not inflated
3. **Independent authorship**: **Must be written or reviewed by someone who did not design the variants**

## Scoring Rubric Design

Specify: dimensions, scale (ratio or interval), aggregation method.

**Important**: If using weighted average, all dimensions must share the same scale type.

**Rubric bias check**: Verify no dimension/weight/phrasing structurally favors one variant.

**Independent review**: The rubric must be reviewed by someone who did not design the variants,
in addition to the Designer's bias check.

## Verification Checklist

- [ ] Tier documented as T3 in experiment log
- [ ] Predicted winning variant pre-registered before evaluators
- [ ] Prompt files verified to exist, contain correct content, and differ
- [ ] Prompt delivered with neutral filename
- [ ] Evaluator instances independent (no shared context)
- [ ] Evaluator model/version/provider recorded and consistent (pinned version ID)
- [ ] Evaluator outputs checked for "file not found"
- [ ] Evaluator did NOT receive ground truth
- [ ] X/Y mapping recorded before scoring
- [ ] Scorer is a different agent instance from evaluators
- [ ] Scorer received de-identified outputs only (no original prompts)
- [ ] Scorer received ground-truth.md and scoring-rubric.md
- [ ] Re-runs capped at ≤ 3 attempts per variant per scenario
- [ ] Per-variant re-run rates compared; if >2× differential, flagged as selection-bias risk
- [ ] ≥ 5 scenarios tested
- [ ] Ground truth balanced (≤ 1/3 variable-specific; ≥ 10 items)
- [ ] GT authored by non-Designer, or reviewed by non-Designer if Designer-authored
- [ ] Rubric bias check performed
- [ ] Rubric reviewed by someone who did not design the variants
- [ ] Results show consistent direction (unanimous n<5; ≥80% n≥5)
- [ ] Magnitude screen passed per scenario
- [ ] P-values cited with pre-registration (no post-hoc caveat needed)
- [ ] De-identification uses deterministic scheme with secret salt
- [ ] Multiple scorers ran; inter-scorer agreement reported
- [ ] If scorers disagreed, adjudication performed and outcome recorded

## Example

```
Designer:
  1. Pre-register predicted winning variant in experiment log
  2. Prepare 5 scenarios with balanced ground truth (≥ 10 items each)
  3. Write variant A and B as self-contained prompts
  4. Verify prompts exist, are non-trivial, and differ
  5. Copy assigned variants to evaluator-prompt.md
  6. Run 2 evaluators per scenario (1×A, 1×B), independent instances, pinned model IDs
  7. De-identify outputs (X/Y with deterministic scheme + secret salt)
  8. Run ≥ 2 scorers (fresh independent instances) with rubric + GT + outputs
  9. Report inter-scorer agreement; adjudicate disagreements
  10. Map scores back to A/B; interpret with direction, magnitude, and p-values
```

## Notes

- Agent instances cannot run sub-instances in most tooling. Launch manually in separate
  sessions.
- LLM instances have variance — this is a feature, not a bug.
- If results contradict across scenarios, conclusion is "no significant difference."
- At T3 stakes, scenario exclusion should be rare (prepare ≥ 6). If multiple scenarios
  are excluded, consider whether the experimental design is too fragile for the claim.
