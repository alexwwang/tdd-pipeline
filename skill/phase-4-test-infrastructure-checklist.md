# Phase 4 Test Infrastructure Checklist

> **Status**: Mandatory gate before Ralph review loop starts.
> **When**: After test code completion, before Ralph review.

---

## ⛔ Why This Exists

Ralph loop uses independent reviewers with clean context. If test code has structural infrastructure gaps (missing fields, wrong constructors, shared state), every reviewer independently re-discovers the same gap, reports it as M/L, and the loop never converges. This is not a review quality problem — it's a **missing compilation step**.

---

## Checklist

```
□ Schema: Every field accessed by tests exists in the type definition
  - grep -rn 'result\.\|state\.\|output\.' test/ --include='*.ts'
  - Verify each accessed property is defined in the corresponding interface/type
  - If not: add optional field to schema (undefined by default)

□ Constructor: Every class instantiated by tests accepts the test's arguments
  - grep -rn 'new [A-Z]' test/ --include='*.ts' | grep -v 'Error\|Map\|Set\|Array'
  - For each `new ClassName(...)`, compare argument count vs constructor signature
  - If mismatch: add parameter to constructor (optional, ignored)

□ Assertions: Each test verifies all relevant output fields for its claim
  - For each test, check assertions cover all fields implied by the test name + plan
  - If missing: add assertion

□ DRY: Cross-file helpers are extracted to shared helpers file
  - grep -rn 'function \|const \|def ' test/ | sort | uniq -d -f2
  - Extract duplicates to shared file, import in consumers

□ Fixture: Each fixture triggers exactly one validation error
  - For each invalid fixture, count distinct errors it would trigger
  - If >1: redesign to isolate the target error
  - Document intentional deviations

□ Plan sync: Test code changes are reflected in the test plan
  - Verify every test in code maps to a plan entry and vice versa
  - Update plan for: renamed/removed/added tests, fixture changes

□ Test Isolation: No test shares mutable state with another test
  - grep -rn 'let \|mutable\|global\|module\.' test/ --include='*.ts' | grep -v 'const \|import\|//'
  - If shared mutable state exists: verify setup/teardown resets it
  - ⛔ Tests that pass in isolation but fail in suite = undetected state pollution

□ Stub Correctness: Every stub causes its consuming test to fail
  - Run full test suite — ⛔ ALL tests MUST fail (TDD RED phase). Any GREEN test = broken stub.
  - grep -rn 'throw\|STUB\|NotImplemented' src/ | grep -v test
    → Stub returning a concrete value instead of throwing may mask a missing test.

□ Mock Return Shape: Mock return values match the planned return type shape
  - For each mock, verify it returns ALL fields in the type definition (not just current test fields)
  - ⛔ Mock returning partial shape passes Phase 4 but explodes in Phase 5
```

## Verification Gate

Produce an evidence summary. **If any item has no evidence citation, the checklist is incomplete.**

```
Item          | Evidence
──────────────┼──────────────────────────────────────────────────────
Schema        | N accesses grep'd, M verified, K fields added
Constructor   | N instantiations, M matched, K params added
Assertions    | N tests checked, M assertions added
DRY           | N duplicates found, M extracted
Fixture       | N fixtures traced, M redesigned
Plan sync     | N tests vs M plan entries, K mismatches fixed
Test Isolation| N mutable vars, M verified reset, K refactored
Stub Correct. | All tests fail: yes/no. N stub fixes.
Mock Shape    | N mocks checked, M fields added
```

## Implementation Notes

**Test isolation**: Shared mutable state between tests causes pass-in-isolation/fail-in-suite or non-deterministic failures. Fix: fresh fixtures per test or explicit reset in setup/teardown.

**Stub correctness**: A stub returning `null`, `0`, `''`, or `[]` instead of throwing means the test is GREEN but tests nothing. Phase 5 implementation won't change the outcome. The TDD RED guarantee is broken.

**Mock return shape**: A mock returning `{status: 'ok'}` when the real type has `{status, data, count}` will pass Phase 4 tests but fail in Phase 5 when business code accesses `.data`. The mock must be a structural stand-in, not a minimal stub.

**Schema/Constructor stubs**: Adding optional fields or parameters to existing types is a type-level stub — no runtime behavior change. Tests compile, but fail at assertion (`undefined !== expectedValue`). This is not business logic.
