---
name: pre-release-testing
description: >
  Pre-release testing and bug root cause analysis for the TDD pipeline's
  validation closure (Phase 6). Covers: Sub-Phase testing (Phase 0–3,
  with conditional Phase 1.5 Soak Test and Phase 1.6 Contract Verification),
  追问 (root cause investigation) protocol with termination guarantees, rollback paths, and Release
  Gate Checklist. On Sub-Phase failure, additionally load
  phase-6-root-cause-investigation.md.
---

# Pre-Release Testing — Phase 6: Validation Closure

**Core Principle**: Evidence over assumptions. Every claim must trace to a concrete observation.

**Verification Diversity Principle**: No single verification method is sufficient. Different methods catch different bug classes:
- **Runtime tests** catch behavior errors
- **Compilation / type-system checks** catch contract mismatches and missing fields
- **Static analysis** catches anti-patterns and unreachable code
- **Expert review** catches design gaps, edge cases, and future risks that automated tools cannot see

A verification plan that relies on only one of these is incomplete. Combine automated verification (fast, repeatable) with expert review (deep, contextual) for comprehensive coverage.

**Scope**: Multi-component systems (stdio, HTTP, message queues). Monoliths: collapse Phase 0+1, retain Layer Isolation.

---

## Part 1: Standard Pre-Release Testing Process

### Phase 0: Component Unit Tests

Run each component's test suite in isolation.

```sh
pytest                    # Python
vitest run                # TypeScript
```

**Gate:** All unit tests pass — zero test failures, errors, or exceptions during test execution.

> Unit tests typically catch ~22% of real bugs. The rest are integration or cross-cutting. Passing this gate is necessary but far from sufficient.

### Phase 1: Integration / E2E Tests

```text
Layer 1: Can the component start standalone? (stdio test, health check)
Layer 2: Can components connect to each other? (integration test)
Layer 3: Does the business logic work across the full chain? (E2E test)
```

**Execute in strict sequence:** Do not proceed to Layer 2 until Layer 1 passes for ALL components.

**Gate:** All integration tests pass. Document known limitations (e.g., "SIGKILL not catchable").

**Watch for:** path-resolution failures (tilde, symlinks, hardcoded paths), config drift between environments, registration gaps (implemented but unwired tools), startup-blocking bugs (stale PID files, dead queues).

### Phase 1.5: Resource Stability (Soak Test)

**When to run:** System has long-running processes, connection pools, file handle management, or in-memory caches.

```text
Pattern: N-iteration resource stability test
  1. Measure baseline: memory (RSS/heap), open FDs, pool connections
  2. Run core operation N times (N ≥ 100, scale by operation cost)
  3. Force GC/cleanup
  4. Assert: growth within tolerance (e.g., <5MB memory, <1 FD, pool returned to baseline)
```

**Language-specific tools:**
- Python: `tracemalloc` + `gc.collect()` + `psutil.Process().num_fds()`
- Node.js: `--expose-gc` + `process.memoryUsage()` + `/proc/self/fd`
- CI flag: gate behind `SOAK_TEST=1` env var to avoid slowing fast test runs

**Gate:** Resource metrics stable across iterations. Zero monotonic growth.

### Phase 1.6: Contract Verification

**When to run:** ≥2 independent deployable components communicating via API/IPC.

```text
Pattern: Consumer-driven contract test
  1. Consumer defines expected API shape (request + response schema)
  2. Generate contract file from consumer test
  3. Provider verifies against real implementation
  4. CI gate: can-i-deploy check before deployment
```

**Lightweight alternative (no Pact Broker):**
- Define shared JSON Schema for each API boundary
- Producer test: output validates against schema
- Consumer test: input validates against same schema
- CI: schema files are single source of truth, any breaking change fails both sides

**Gate:** All contracts verified. No schema drift between consumer expectations and provider output.

### Phase 2: Cross-Cutting Validation

| Dimension | What to check | How |
|-----------|---------------|-----|
| Config consistency | Same paths, ports, env vars across all config files | grep for hardcoded paths, tilde, localhost |
| CI reproducibility | Tests pass in CI, not just locally | Run CI pipeline |
| Regression guard | Previously fixed bugs haven't returned | Run regression test suite |
| Format / lint | Code style consistent | `ruff format --check`, `ruff check` |
| Build artifact | Build output is correct and deployable | Build + deploy to staging |
| Doc accuracy | Docs match current state | Verify test counts, paths, commands |

**Gate:** All checks pass. Document deviations.

### Phase 3: Manual / Exploratory Validation

Catches bugs requiring subjective judgment (comprehensibility of messages, appropriateness of error handling, edge cases tests didn't cover).

- Passive trigger detection (does the system activate when it should?)
- False positive check (does it activate when it shouldn't?)
- User-visible feedback (completion notifications, error messages)
- Edge cases tests didn't cover

**Gate:** Manual checklist all ✅ or documented as known limitation.

### Release Gate Checklist

- [ ] All automated tests pass (unit + integration + E2E + regression)
- [ ] CI pipeline green
- [ ] No hardcoded paths / credentials in tracked files
- [ ] Config files consistent across environments
- [ ] Previously fixed bugs verified non-regressed
- [ ] Documentation accurate (test counts, paths, commands)
- [ ] User-visible behaviors manually verified
- [ ] Known limitations documented
- [ ] Resource stability verified (soak test passed, no leaks) *(required only if system has long-running processes, connection pools, or caches — see Phase 1.5 "When to run" for applicability)*
- [ ] API contracts verified (no schema drift between components) *(required only if ≥2 independent deployable components communicate via API/IPC — see Phase 1.6 "When to run" for applicability)*
- [ ] Fault tolerance tested (non-critical dependency failure degrades gracefully)
- [ ] Error handler coverage (every catch block tested)
- [ ] No retry amplification (retry budget bounded under failure injection)

### ⛔ N/A Evidence Rule

Marking any checklist item as N/A **requires negative evidence** — proof that the condition does not exist in this codebase. The evidence column must contain:

1. **What the item checks** (the specific condition or pattern)
2. **Why the condition is absent** (grep command with 0 matches, architectural argument, or code analysis)
3. **The conclusion** (N/A because X was verified absent)

N/A without evidence is the checklist equivalent of re-labeling H issues as "accepted design deviations" to bypass the gate — it lowers the bar to avoid doing the work.

| ❌ WRONG N/A | Why It's Wrong | ✅ CORRECT |
|-------------|----------------|-----------|
| "Not a long-running process" → N/A for resource stability | In-memory caches, growing arrays, and Maps can leak in any process | Grep for all growable data structures, verify each has bounds or eviction |
| "No retry logic" → N/A without checking | Assumption without evidence | `grep -rn 'retry\|reconnect\|backoff\|attempt' src/` → report 0-match evidence |
| "Library, not a service" → N/A for multiple items | Being a library doesn't exempt it from checks | Each item's applicability must be independently assessed with evidence |

---

## Part 2: Bug Root Cause Investigation

> **Extracted for progressive disclosure.** Load **`phase-6-root-cause-investigation.md`** when a sub-phase fails. Contains: Layer Isolation (Q1–Q4), Evidence Collection (Q5–Q7), 5-Why Drill, Fix Verification (V1–V5), and 12 common bug patterns.

---

## Part 3: Integration Gap Detection Checklist

For each pair of interacting components, walk through every row. If any cell is "we don't know," that's your gap.

*(The same 14 dimensions appear in phase-7-system-quality-audit.md Part 1. Phase 7 will skip its dimension check if Phase 6 already covered all pairs. Keep both lists in sync.)*

| Gap | Question | Evidence to collect |
|-----|----------|-------------------|
| **Schema** | Do components agree on data formats? | Compare input/output schemas at each boundary |
| **State** | Is state managed correctly across process boundaries? | Check: who creates, who reads, who cleans up temp files |
| **Timing** | Are there race conditions between components? | Check: startup ordering, idle detection, polling intervals |
| **Error propagation** | Does an error in component A surface in component B? | Inject an error in A, verify B detects and handles it |
| **Config propagation** | Does the same config reach all components? | Compare resolved config (not config files) at each component |
| **Registration chain** | Is every service discoverable by its consumers? | Enumerate registered tools/services, compare with expected |
| **Lifecycle** | Does shutdown clean up everything startup creates? | Kill process, check for orphaned files/processes |
| **Freshness** | Does the system work on a clean slate AND with residual state? | Test with and without `rm -rf` of temp dirs |
| **Serialization** | Do cross-language components handle edge cases (precision, encoding, null semantics) consistently? | Shared test corpus: serialize in Lang A → deserialize in Lang B → re-serialize → compare |
| **Error handler** | Does each catch block actually work? | Inject specific error → assert correct handling (no swallow, no amplification, cleanup ran) |
| **Timeout chain** | Is timeout propagation correct through the dependency chain? | Verify caller timeout < callee timeout at each hop; inject latency > caller timeout, assert fast fail |
| **Retry budget** | Does retry amplification stay bounded when a dependency fails? | Inject 100% error rate → verify upstream request count ≤ normal × (1 + retry_budget) |
| **Concurrency** | Is shared mutable state safe under concurrent access? | N threads hit shared state simultaneously, assert invariants hold (counter = expected, no duplicates) |
| **Performance logic** | Do hot paths handle realistic data volume? | Test with N > 1 items; verify batch operations don't trigger per-item queries; assert latency within budget |

---

## Part 4: Pipeline Integration

Phase 6 is the pipeline's **validation closure** — it validates Phases 1–5 output through systematic testing. It does NOT use Ralph loop. Phase 6 uses sub-phase gates and 追问 protocol instead of reviewer subagents.

### Quality Mechanisms

| Mechanism | When | Purpose |
|-----------|------|---------|
| **Sub-Phase gates** | Phase 0–3 (0 → 1 → 1.5? → 1.6? → 2 → 3) | Objective pass/fail verification |
| **追问 protocol** | Sub-Phase failure | Root cause investigation with termination guarantees |
| **Release Gate Checklist** | All Sub-Phases pass | Evidence compilation for Phase 7→8→user decision |

### Sub-Phase Execution Rules

1. **Strict sequential**: Phase 0 → 1 → 1.5 (if applicable) → 1.6 (if applicable) → 2 → 3. No skipping.
2. **Gate is pass/fail**: No severity classification. Green or red.
3. **Any failure** → load `phase-6-root-cause-investigation.md`, run 追问 protocol → determine rollback target.
4. **All pass** → fill Release Gate Checklist with evidence → proceed to Phase 7.

### 追问 (Root Cause Investigation) Protocol Summary

```
Sub-phase fails
  → Layer Isolation (Q1→Q4): locate broken layer
  → Evidence Collection (Q5→Q7): gather facts before hypothesizing
  → 5-Why Drill (≤5 levels, ≤3 branches, every level needs evidence)
  → Termination check (T1+T2+T3 all required):
       T1. Actionability: maps to specific implementable fix
       T2. Falsifiability (two sub-checks):
           T2a. Sufficiency: "If we implement [prevention], would bug disappear?" → Yes
           T2b. Specificity: "If real cause were [most plausible alternative], would [prevention] still fix it?" → No
           (T2b skippable only when no plausible alternative exists; must document why)
       T3. Explanatory power: root cause explains ALL observed symptoms
  → Independent confirmation (one person/agent, T2a+T2b check, no loop)
  → Root cause confirmed → enter rollback path
```

**Hard constraints:** HC1 depth≤5, HC2 branches≤3, HC3 evidence anchoring, HC4 layer scope (code/arch/config/process only). Independent confirmation required (CONFIRM→fix, CHALLENGE→resume with new evidence, no counter reset). Full definitions in `phase-6-root-cause-investigation.md`.

### Rollback Paths

Root cause directly determines rollback target (T1 guarantees actionability, no ambiguity):

| Root Cause Layer | Rollback To | Re-run Scope |
|-----------------|-------------|-------------|
| Test gap | Phase 4 | Phase 4 → 5 → 6 → 7 → 8 |
| Code bug | Phase 5 | Phase 5 → 6 → 7 → 8 |
| Design flaw | Phase 2 | Phase 2 → 3 → 4 → 5 → 6 → 7 → 8 |
| Requirement misunderstanding | Phase 1 | Full pipeline |
| Config/environment only | Fix config | Phase 6 only (full re-run), then 7 → 8 |

**Rules:** (1) Phase 6 rerun is always full from Phase 0, never incremental. (2) Rollback to Phase 1–3 preserves existing code but requires new Ralph loops. (3) Config-only fixes need no code change.

### Next Phase

When all sub-phases pass, load **`phase-7-system-quality-audit.md`** for incremental system-level quality review (16-pattern catalog, integration pair discovery, execution-order analysis). Phase 7 finds issues that Phase 6's testing-focused analysis may miss.
