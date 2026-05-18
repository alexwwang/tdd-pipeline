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

## Part 3: Common Bug Patterns

Based on 18 bugs from real projects. **⛔ Systematically check ALL 16 patterns — ruling out non-matches is as important as confirming matches.**

### Pattern 1: Path & Config Issues (5/18)
Hardcoded paths, tildes in config, symlinks not resolved. Also check: SQL string interpolation with unquoted column/table names (`", ".join(cols)` without quoting), env vars used without default or validation.

### Pattern 2: Registration / Wiring Gaps (3/18)
Exported but not registered = invisible at runtime. Compare exported symbols vs registered tools/services.

### Pattern 3: Initialization / Startup Blocking (2/18)
Verify fails fast, not hangs. Check for init-phase awaits without timeout, stale PID/lock/marker files.

### Pattern 4: Silent Failures / Missing Feedback (2/18)
Non-interactive errors logged at debug/info instead of warn+. Background operations missing completion notification.

### Pattern 5: Test–Production Parity Gaps (2/18)
Bugs in production but invisible to tests happen when test harness uses a different activation path than real users.

### Pattern 6: Integration Bugs (4/18)
Only manifest when two components interact. Check: subprocess IPC (use spawn, not exec), silently ignored params, IPC limits, ID propagation.

### Pattern 7: Resource Leaks (Memory, FDs, Connections)
System degrades over time. Run N-iteration soak test → assert no monotonic growth in memory, FDs, connections.

### Pattern 8: Race Conditions — Concurrent Access & Startup Races
Flaky tests, intermittent wrong results, ghost processes. Check: shared mutable state synchronized? PID/lock file creation atomic?

### Pattern 9: Serialization Boundary Bugs

Cross-language edge cases — test each boundary:

| Risk | Test Input | Expected Behavior |
|------|-----------|-------------------|
| Integer > 2^53 | `9007199254740993` | Serialized as string, not number |
| Null vs missing | `{"a": null}`, `{}`, `{"a": ""}` | Consumer distinguishes all three |
| Unicode beyond BMP | `"🚀🎉"` | Round-trip preserves bytes |
| Float precision | `0.1 + 0.2` | Assert approximate, not exact |
| Date timezone | `"2024-01-01T00:00:00+05:00"` | Consumer interprets timezone correctly |

### Pattern 10: Version Drift — Interface Contract Breaking
Component A upgraded its API, component B still calls the old interface. Distinct from integration bugs (P6): P6 is about initial mismatch, this is about drift over time. Check: are component versions pinned? Do consumer tests verify against actual provider (not mocks)?

### Pattern 11: Cascading Failures & Retry Storms
Inject 100% error rate → verify retry budget bounded (upstream requests ≤ normal × (1 + retry_budget)), circuit breaker opens, graceful degradation, auto-recovery.

### Pattern 12: Auth / Permission Boundary Inconsistency
Components enforce different access control policies — one checks permissions, another trusts internal calls unconditionally. Check: is auth verified at every boundary, or only at the edge?

### Pattern 13: Error Handling Bugs — Handlers That Introduce Bugs
Check every error path for three dimensions: (1) catch/except blocks tested — inject exact exception, assert context preserved and cleanup ran; (2) return value semantics — does the function return what it claims (actual DB rows, not input count)? (3) schema consistency — do success and failure paths return the same shape (missing columns on error = downstream KeyError)?

### Pattern 14: Performance Logic Defects — N+1, Slow Paths, Batch Misrouting
Test with N>1 items → assert O(N) growth not O(N²), verify batch API used for batch operations (≥5× faster than single-item calls).

### Pattern 15: Implicit Contract Violations
Undocumented semantic assumptions broken at component boundaries: call ordering (must init before query), sync vs async expectations, thread-safety assumptions. Distinct from P6 (explicit interface mismatch) — these are assumptions never written down.

### Pattern 16: Data Validation Logic Defects
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
