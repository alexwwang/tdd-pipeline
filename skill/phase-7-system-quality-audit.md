---
name: system-quality-audit
description: >
  System-level quality audit with 16 bug patterns (grep-powered), integration
  pair discovery, and execution-order analysis. Loaded after Phase 6 completes
  as an incremental second pass. Receives Phase 6 results and finds issues
  missed by the Phase 6 audit.
---

# System Quality Audit — Pattern Catalog & Integration Analysis

**Core Principle**: Systematic coverage over intuition. Grep commands find what reading misses.

**When loaded:**
- After Phase 6 completes (triggered by SKILL.md progressive disclosure)
- Runs as an independent second-pass audit against the same source code
- Receives Phase 6 results to avoid duplicating findings

---

## Part 1: Pair Discovery

Before scanning patterns, enumerate ALL interacting component pairs. Missing a pair means undetected bugs.

### Pair Types (check each)

| Pair Type | How to Identify | Example |
|-----------|----------------|---------|
| **Direct neighbors** | A calls B's API directly | adapter → repository |
| **Indirect data flow** | A writes data that B reads, through an intermediary or shared store | adapter → DuckDB file → enrichment query |
| **Lifecycle coupling** | A creates/manages a resource that B depends on | connection pool → all queries |
| **Test ↔ Production** | Test harness exercises a different code path than real users | mock trigger vs actual stdin |

**⛔ Rule**: If a component pair exists in the system but is NOT in your list, that's an undetected integration gap.

### Dimension Check Per Pair

For EACH pair, walk through ALL 14 dimensions. Skipping dimensions means undetected gaps.

| Dimension | Question | Evidence |
|-----------|----------|----------|
| Schema | Do components agree on data formats? | Compare input/output schemas at boundary |
| State | Is state managed correctly across boundaries? | Who creates, reads, cleans up temp files |
| Timing | Race conditions between components? | Startup ordering, polling intervals |
| Error propagation | Does error in A surface in B? | Inject error in A, verify B detects it |
| Config propagation | Same config reaches all components? | Compare resolved config at each component |
| Registration chain | Every service discoverable by consumers? | Enumerate registered vs expected |
| Lifecycle | Shutdown cleans up everything startup creates? | Kill process, check orphans |
| Freshness | Works on clean slate AND with residual state? | Test with/without cleanup |
| Serialization | Cross-language edge cases handled? | Shared test corpus round-trip |
| Error handler | Each catch block actually works? | Inject error → assert handling |
| Timeout chain | Timeout propagation correct? | Caller timeout < callee timeout at each hop |
| Retry budget | Retry amplification bounded? | 100% error → verify bounded upstream requests |
| Concurrency | Shared mutable state safe? | N threads, assert invariants |
| Performance logic | Hot paths handle realistic volume? | N > 1 items, assert linear growth |

---

## Part 2: 16 Bug Pattern Catalog

Based on 18 bugs from real projects. **⛔ Systematically check ALL 16 patterns — ruling out non-matches is as important as confirming matches.**

> Adapt grep paths/extensions to your project. The categories are universal even if paths are not.

### Pattern 1: Path & Config Issues (5/18)

```sh
grep -rn '/Users/\|/home/\|C:\\' --include='*.ts' --include='*.py' --include='*.json' .
grep -rn '~/' --include='*.json' --include='*.yaml' --include='*.toml' .
grep -rn 'join.*cols\|", "\.join\|format.*column' --include='*.py' --include='*.ts' .  # SQL unquoted identifiers
```

Hardcoded paths, tildes in config, symlinks not resolved. Also check: SQL string interpolation with unquoted column/table names (`", ".join(cols)` without quoting), env vars used without default or validation.

### Pattern 2: Registration / Wiring Gaps (3/18)

```sh
grep -rn 'export function\|export class\|def ' src/ | grep -v test    # exports
grep -rn 'register\|\.tool(\|mcp\.tool(' src/init.ts                    # registrations
```

Exported but not registered = invisible at runtime. Compare exported symbols vs registered tools/services.

### Pattern 3: Initialization / Startup Blocking (2/18)

```sh
time <start-command>                                                    # should be < 5s
grep -rn 'await\|fetch\|connect' src/init.ts | grep -v 'timeout'       # no timeout protection
grep -rn 'marker\|\.pid\|\.lock\|\.active' src/                         # stale state cleanup?
```

Verify fails fast, not hangs. Simulate unavailable dependency → verify fails fast, not hangs.

### Pattern 4: Silent Failures / Missing Feedback (2/18)

```sh
grep -rn 'notify\|toast\|alert\|callback' src/ --include='*.ts' | grep -v test
grep -rn 'logger\.\(debug\|info\)' src/ | grep -v test
```

Non-interactive errors must use warn/error, not debug/info. Every background operation needs completion notification.

### Pattern 5: Test–Production Parity Gaps (2/18)

```sh
grep -rn 'send-keys\|stdin\|mock.*trigger' test/ | head -5
grep -rn 'SIGKILL\|kill -9' test/
grep -rn 'beforeEach\|beforeAll\|setUp\|rm -f' test/ | grep -c 'marker\|\.active'
```

Bugs in production but invisible to tests happen when test harness uses a different activation path than real users.

### Pattern 6: Integration Bugs (4/18)

```sh
grep -rn 'execFile\|execSync' src/ && echo "Warning: use spawn for bidirectional IPC"
grep -rn 'agent:\|model:' src/ | grep -v test          # unsupported params silently ignored?
grep -rn 'parentId\|sessionId\|ownerId' src/ | grep -v test  # IDs propagated, not hardcoded?
```

Only manifest when two components interact — invisible to single-component tests. Check: subprocess IPC, silently ignored params, IPC limits, ID propagation.

### Pattern 7: Resource Leaks — Memory, FDs, Connections

```sh
lsof -p $PID | wc -l   # FDs: before and after
grep -rn 'pool\|connection\|client' src/ --include='*.py' --include='*.ts' | grep -v test | grep -c 'close\|release\|disconnect'
# Low close/release count = likely leak
```

```python
# Memory: N-iteration growth check
import tracemalloc, gc
tracemalloc.start(); gc.collect()
before = tracemalloc.get_traced_memory()
for _ in range(1000): operation_under_test()
gc.collect()
after = tracemalloc.get_traced_memory()
growth_kb = (after[0] - before[0]) / 1024
assert growth_kb < 100, f'Leaked {growth_kb}KB'
```

System degrades over time. N-iteration soak test → assert no monotonic growth in memory, FDs, connections.

### Pattern 8: Race Conditions — Concurrent Access & Startup Races

```sh
grep -rn '\.pid\|\.lock\|\.active' src/ | grep -v test
# Check: is creation atomic? (O_CREAT | O_EXCL, not check-then-create)
grep -rn 'global\|static\|\.cache\|\.state' src/ --include='*.py' --include='*.ts' | grep -v test | grep -v 'const '
# Each hit: is access synchronized?
```

**Test pattern:**
```text
# N threads × M iterations, assert invariant
spawn N workers (ThreadPoolExecutor / thread pool), each performing shared_operation
assert invariant_holds(results)  # e.g., counter == expected, no duplicates
```

Flaky tests, intermittent wrong results, ghost processes.

### Pattern 9: Serialization Boundary Bugs — Cross-Language & Edge Cases

```sh
grep -rn 'JSON\.\(parse\|stringify\)\|json\.loads\|json\.dumps\|serialize\|deserialize' src/ | grep -v test
```

| Risk | Test Input | Expected Behavior |
|------|-----------|-------------------|
| Integer > 2^53 | `9007199254740993` | Serialized as string, not number |
| Null vs missing | `{"a": null}`, `{}`, `{"a": ""}` | Consumer distinguishes all three |
| Unicode beyond BMP | `"🚀🎉"` | Round-trip preserves bytes |
| Float precision | `0.1 + 0.2` | Assert approximate, not exact |
| Date timezone | `"2024-01-01T00:00:00+05:00"` | Consumer interprets timezone correctly |

### Pattern 10: Version Drift — Interface Contract Breaking

```sh
grep -rn 'version\|pin\|lock' package.json requirements.txt pyproject.toml  # are deps pinned?
grep -rn 'mock\|Mock\|stub' test/ | grep -c 'import\|from'                  # how many tests mock providers?
```

Component A upgraded its API, component B still calls the old interface. Distinct from P6 (initial mismatch) — this is drift over time. Check: are component versions pinned? Do consumer tests verify against actual provider (not mocks)?

### Pattern 11: Cascading Failures & Retry Storms

```sh
grep -rn 'retry\|backoff\|maxAttempt\|circuit' src/ config/ --include='*.py' --include='*.ts' --include='*.json' --include='*.yaml'
# Warning signs: no backoff, no jitter, no retry budget, no circuit breaker
grep -rn 'timeout\|Timeout\|TIMEOUT' src/ config/ | grep -v test
# For each A→B call: verify timeout_A ≤ timeout_B
```

**Test pattern:**
```text
1. Inject 100% error rate on dependency X (via Toxiproxy or mock)
2. Assert: upstream request count ≤ normal × (1 + retry_budget)
3. Assert: circuit breaker opens within N failures
4. Assert: core functionality degrades gracefully (fallback works)
5. Restore dependency → assert system recovers automatically
```

### Pattern 12: Auth / Permission Boundary Inconsistency

```sh
grep -rn 'auth\|permission\|token\|session' src/ --include='*.py' --include='*.ts' | grep -v test
# For each component boundary: is auth verified, or does it trust internal calls?
```

Components enforce different access control policies — one checks permissions, another trusts internal calls unconditionally. Check: is auth verified at every boundary, or only at the edge?

### Pattern 13: Error Handling Bugs — Handlers That Introduce Bugs

```sh
# Find catch/except blocks without test coverage
grep -rn 'catch\|except\|try' src/ --include='*.py' --include='*.ts' | grep -v test | wc -l
# Count should match (approximately) test count targeting error paths

# Find empty catch blocks (silent swallow)
grep -A2 'catch\|except' src/ --include='*.py' --include='*.ts' | grep -v test | grep -E '^\s*(pass|}|return|//|#$)'
# Each hit is a potential silent failure
```

**Error handler test checklist (per catch/except block):**
```text
- Test injects the exact exception type the catch handles
- Asserts error context preserved (cause chain, error code, relevant IDs)
- Asserts cleanup/finally ran (no resource leak on error path)
- Asserts system is usable after error recovery
- If retry: assert retry classification (retryable vs non-retryable)
- If fallback: assert fallback output is valid, not just "no crash"
```

Check every error path for three dimensions: (1) catch/except blocks tested, (2) return value semantics — does the function return what it claims (actual DB rows, not input count)? (3) schema consistency — do success and failure paths return the same shape?

### Pattern 14: Performance Logic Defects — N+1, Slow Paths, Batch Misrouting

```sh
# N+1 detection: loops containing async/DB calls
grep -rn 'for\|while\|\.forEach\|\.map' src/ --include='*.py' --include='*.ts' | grep -v test | grep -B1 -A1 'await\|fetch\|\.query\|\.execute'
# Each hit: is this a loop-per-item query? → N+1 candidate

# Batch misrouting: single-item path used for batch operations
grep -rn 'batch\|bulk\|createMany\|insert_all' src/ --include='*.py' --include='*.ts' | grep -v test
grep -rn 'for.*in\|\.forEach' src/ --include='*.py' --include='*.ts' | grep -v test | grep -c 'await\|create\|insert\|save'
# High loop count + low batch usage = likely misrouting
```

**Test pattern:**
```text
1. Prepare N items (N > 10, ideally realistic production volume)
2. Run batch operation, measure latency
3. Assert: latency grows linearly O(N), not quadratically O(N²)
4. Compare: N items via batch API vs N items via single-item calls
5. Assert batch path is ≥5× faster for same N
```

### Pattern 15: Implicit Contract Violations

```sh
grep -rn 'init\|setup\|configure\|start' src/ --include='*.py' --include='*.ts' | grep -v test | head -10
# Are there call-ordering requirements? (must init before query)
grep -rn 'async def\|await\|asyncio\|Promise' src/ | grep -c 'sync\|Sync\|blocking'
# Mixing sync/async expectations at boundaries?
```

Undocumented semantic assumptions broken at component boundaries: call ordering (must init before query), sync vs async expectations, thread-safety assumptions. Distinct from P6 (explicit interface mismatch) — these are assumptions never written down.

### Pattern 16: Data Validation Logic Defects

```sh
grep -rn 'assert\|validate\|check\|sanity\|threshold\|clip\|clamp\|nullif' src/ --include='*.py' --include='*.ts' | grep -v test
# For each: does validation nullify the entire row, or only the offending field?
grep -rn 'fillna\|isna\|isnull\|dropna\|np.nan' src/ --include='*.py' | grep -v test
# Check: does validation failure cascade to unrelated columns?
```

Validation rules that are too aggressive (nullifying valid data) or too permissive (accepting garbage). Check: does the validation threshold have a sound basis? Could legitimate outliers be caught? Does a failing validation cascade to unrelated data (e.g., nullifying entire row for one bad column)?

---

## Part 3: Execution Order Analysis

A special case of P16 that deserves its own check: **validation functions that short-circuit**.

### The Execution Order Bug Pattern

When multiple validation stages run in sequence, an early `return` or `raise` can prevent more precise downstream checks from ever executing.

**Detection:**
```sh
grep -rn 'return\|raise\|break' src/ --include='*.py' --include='*.ts' | grep -B5 'valid\|check\|sanity\|assert'
# Each early exit: does it prevent a more granular check from running?
```

**Test pattern:**
```text
1. Find all validation stages in sequence
2. For each stage: craft input that fails THIS stage but would pass later stages
3. Assert: later stages STILL execute (or their results are available)
4. If early return exists: assert it returns per-field results, not "all bad"
```

**Why this matters:** A median sanity check that nullifies all fields and returns early prevents a per-stock check from distinguishing "this one stock has garbage data" from "all stocks are fine." The result: all stocks lose data, not just the bad one.

---

## Part 4: Verification Audit

Phase 7 output must pass an independent verification pass before acceptance. This is NOT a Ralph loop — it is a precision check on the Phase 7 findings themselves.

### When to Run

After the main agent completes Parts 1–3 and produces a findings report.

### Who Runs It

An independent subagent (oracle or dedicated verifier) that was NOT involved in producing the Phase 7 findings.

### What to Verify

The verifier receives the Phase 7 findings report and the source code. Verify each finding independently (real? not duplicate? severity accurate? actionable?) and check overall coverage:

- **All 16 patterns scanned**: Does the report show evidence (grep output or "no matches found") for every pattern in Part 2?
- **All pairs analyzed**: Does Part 1 list every component pair, and does every pair have dimension-check results?
- **Execution order checked**: Does Part 3 address every validation chain in the codebase?
- **Evidence present**: Each finding cites specific file:line or grep output — not just descriptions

### Gate Condition

Phase 7 passes when:
- All coverage checks show complete (or gaps are explicitly acknowledged and justified)
- Zero findings remain at C/H that are unaddressed
- False positive rate is documented

**⛔ Findings rejected by the verifier are removed from the report. They do NOT need to be fixed. Downgraded findings retain their new severity.**
