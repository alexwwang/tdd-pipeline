# Bug Root Cause Investigation — 追问方法 & Common Patterns

Loaded when a Phase 6 sub-phase fails. Not loaded during all-pass execution.

---

## Part 2: 追问方法 (Root Cause Investigation)

> **If your first fix didn't work, stop fixing.** Skip to "When a fix attempt fails" below — you almost certainly have the wrong layer.

### Step 1: Layer Isolation (分层定位)

Ask IN ORDER. Stop when you find the broken layer.

```text
Q1: Can the failing component start standalone?
    → Run it in isolation. If it can't start → initialization bug (import errors, missing config, path issues, version mismatch)

Q2: Can it connect to its dependencies?
    → Check connection/auth/network. If it starts but can't connect → integration glue bug (wrong URLs, expired tokens, firewall, config mismatch)

Q3: Does the simplest business case work?
    → Send one minimal request. If even that fails → core logic bug (null handling, type mismatch, off-by-one, wrong assumption)

Q4: Does the full real-world flow work?
    → Only if Q1-Q3 pass. If not → edge case bug (timing, concurrency, resource exhaustion, state accumulation)
```

### Step 2: Evidence Collection (证据收集)

Before hypothesizing, collect evidence.

```text
Q5: What does the log say at the exact moment of failure?
    → No log? Add one. Absence of evidence = absence of logging.

Q6: What was the system's state just before failure?
    → Check config, database, filesystem, env vars. What changed from "known working"?

Q7: Is this fresh state or residual state from a previous run?
    → Check temp files, marker files, caches, PID files.
```

**Evidence hierarchy (most → least reliable):**
1. Source code + runtime log together
2. Runtime log alone
3. Source code analysis alone
4. User description of behavior
5. Theory about what "should" happen

If Layer Isolation fails after two full Q1–Q4 passes, bring in a second pair of eyes. If 5-Why reaches "don't know" at any level → missing instrumentation → add logging, reproduce, then resume. If a bug spans layers (root in Layer 1, symptom in Layer 3), treat as two separate bugs: (1) fix the root cause, (2) fix the error-propagation gap that hid the symptom.

### Step 3: 5-Why Root Cause Drill

Each answer must cite evidence, not speculation. Stop when you reach a systemically preventable cause — this may be at depth 3, 4, or 5.

```text
Why #1: Why did the test fail / user see the bug? → [evidence-based answer]
Why #2–#5: Why did [previous] happen? → [evidence-based, stop at preventable cause]
```

**Example (tilde path bug):**
```text
Why1: MCP server failed → ModuleNotFoundError
Why2: Python fell back to 3.8 (missing modules)
Why3: uv run --project ~/path doesn't expand ~
Why4: Config tested with expanded paths, committed with tilde
Why5: No config validation for non-absolute paths → FIX: add validation
```

### Step 4: Fix Verification (V1→V5)

```
V1: Specific failing test → red to green?
V2: Broader component test suite → still pass?
V3: Full integration/E2E chain → pass?
V4: Full test suite (unit + integration + regression) → no new failures?
V5: Regression test added → same bug caught if it returns?
```

---

## Part 3: Common Bug Patterns & Quick Checks

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

## Hard Constraints (HC1–HC4)

```
HC1. Depth limit: 5 Why levels maximum (ceiling, not target).
     Reached level 5 without T1-T3 → evidence insufficient → stop, add logging/telemetry, reproduce, then resume.

HC2. Branch limit: 3 independent investigation chains per bug maximum.
     Chain 1 fails T2/T3 → try chain 2 with different direction.
     Chain 2 fails → try chain 3 with different framing.
     Chain 3 also fails → escalate to user with evidence gathered.
     T2b alternative tested per branch comes from hypotheses already considered-but-rejected in that chain (no extra divergence).

HC3. Evidence anchoring: Every Why answer must cite concrete evidence.
     "I think it might be..." → invalid. Must cite log/code/config line.
     2 consecutive Whys without new evidence → stop, collect evidence first.

HC4. Layer scope: Root cause must fall in one of:
     - Code implementation (function/module level)
     - Architecture design (component interaction level)
     - Configuration/environment (deployment level)
     - Process/tooling (CI/testing level)
     Root cause at "developer cognition" or "team culture" level → out of technical scope, document and escalate to user.
```

## Independent Confirmation

After 5-Why produces a root cause conclusion, one independent confirmation (no loop):

```
Input: root cause statement + evidence chain + T1-T3 verification

Independent confirmer (NOT involved in the investigation):
  1. Read root cause statement and evidence chain
  2. Identify the most plausible alternative explanation (from hypotheses already considered during investigation)
  3. Independently execute T2a (sufficiency) and T2b (specificity against that alternative)
  4. Conclusion:
     CONFIRM  → proceed to fix → enter rollback path
     CHALLENGE → challenge reason becomes new evidence → resume 5-Why
                 (hard constraints remain — does NOT reset counters)
```

Purpose: break confirmation bias. Only once because if one challenge + continued investigation still can't reach conclusion, escalate to user.

---

## Investigation Workflow

### When investigating a bug:

1. **Determine which phase the bug was found in** (Phase 0/1/2/3)
2. **Run Layer Isolation** (Step 1) → find broken layer
3. **Collect Evidence** (Step 2) before hypothesizing
4. **Run 5-Why Drill** (Step 3) → reach root cause
5. **Classify the bug pattern** (Part 3) → match a known pattern? Add its Quick Check to CI for recurrence prevention.
6. **Verify the fix** (Step 4) in strict V1→V5 sequence

---

## When a Fix Attempt Fails

This is the most important section. If your first fix didn't work:

1. **Stop fixing. Start investigating.** You have the wrong layer.
2. **Re-run Layer Isolation** from the layer *below* where you stopped. If you stopped at Q2, start at Q3.
3. **Check patterns above** — is the real bug a different pattern than you assumed?
4. **Check residual state** — leftover state from previous run?
5. **Check config consistency** — fix correct but deployed to wrong config file?

The #1 debugging mistake: wrong layer. The #2: fixing symptom instead of root cause. This methodology prevents both.
