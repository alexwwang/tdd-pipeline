# Severity Migration Guide (Pre-v0.13 → v0.13+)

## What Changed

Pre-v0.13 used a flat 5-level severity system. v0.13 restructured into a 3-tier, 6-level system.

The old M (Major) was split into M1 (Major-Defect) and M2 (Major-Improvement), and all levels were organized into tiers by cardinality (finite vs infinite) and stop-condition relevance.

## Mapping Table

| Pre-v0.13 | v0.13+ | Tier | Counted in Stop Condition |
|-----------|--------|------|---------------------------|
| C (Critical) | **C** (Critical) | Defect | ✅ Yes |
| H (High) | **H** (High) | Defect | ✅ Yes |
| M (Major) — if behavioral defect | **M1** (Major-Defect) | Defect | ✅ Yes |
| M (Major) — if improvement suggestion | **M2** (Major-Improvement) | Quality | ❌ No |
| L (Low) | **L** (Low) | Quality | ❌ No |
| I (Info) | **I** (Info) | Observation | ❌ No |

## Classification Rules

Old M findings are reclassified using the behavioral heuristic:

> **"If I fix this, will the code do something differently at runtime?"**
> - Yes → M1 (Defect Tier)
> - No, just organized differently → M2 (Quality Tier)

### Common M → M1 cases (Defect)
- Logic error producing wrong result
- Function doesn't match its documented contract
- Missing assertion for a stated requirement
- Type safety gap causing potential runtime error
- Edge case not handled

### Common M → M2 cases (Quality)
- Extract magic number into named constant
- Reduce function parameter count
- Rename for clarity
- Consolidate duplicated logic across files
- Add defensive guard for theoretically-impossible state

## Stop Condition Change

| | Pre-v0.13 | v0.13+ |
|---|-----------|--------|
| **Stop condition** | 2 consecutive rounds with zero new C/H/M | 2 consecutive rounds with zero new C/H/M1 |
| **L resets counter?** | Yes (pre-v0.12) / No (v0.12) | No |
| **M2 resets counter?** | N/A (didn't exist) | No |
| **Round cap?** | None | None |

## Impact on Existing Documents

- **Review logs**: Old logs with `M: N` tallies are valid. Map to `M1 + M2 = N` based on the classification of each finding.
- **GPAV submissions**: Old submissions with `severity: 'M'` remain valid for historical records. New submissions must use `'M1'` or `'M2'`.
- **Known Issues documents**: Old KI entries with severity `M` should be re-evaluated at next periodic review (every 3 rounds) and reclassified as M1 or M2.
- **Review prompts (review-design.md, review-code.md)**: Output format already updated to `C|H|M1|M2|L|I`.
