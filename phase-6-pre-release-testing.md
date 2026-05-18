---
name: pre-release-testing
description: >
  Pre-release testing and bug root cause analysis for the TDD pipeline's
  validation closure (Phase 6). Covers: sub-phase testing (Phase 0–3,
  with conditional Phase 1.5 Soak Test and Phase 1.6 Contract Verification),
  追问 protocol with termination guarantees, rollback paths, and user
  go/no-go decision. On sub-phase failure, additionally load
  phase-6-root-cause-investigation.md.
---

# Pre-Release Testing — Phase 6: Validation Closure

**Core Principle**: Evidence over assumptions. Every claim must trace to a concrete observation.

**Scope**: Multi-component systems (stdio, HTTP, message queues). Monoliths: collapse Phase 0+1, retain Layer Isolation.

---

## Part 1: Standard Pre-Release Testing Process

### Phase 0: Component Unit Tests

Run each component's test suite in isolation.

**Gate:** All unit tests pass. Zero exceptions.

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

Pattern: measure baseline (memory/FDs/pool) → run core operation N≥100 iterations → force GC/cleanup → assert growth within tolerance. Gate behind `SOAK_TEST=1` env var in CI.

**Gate:** Resource metrics stable across iterations. Zero monotonic growth.

### Phase 1.6: Contract Verification

**When to run:** ≥2 independent deployable components communicating via API/IPC.

Use JSON Schema at each API boundary as single source of truth — producer and consumer tests validate against the same schema. No Pact Broker needed.

**Gate:** All contracts verified. No schema drift between consumer expectations and provider output.

### Phase 2: Cross-Cutting Validation

Check: config consistency, CI reproducibility, regression guard, format/lint, build artifact, doc accuracy.

**Gate:** All checks pass. Document deviations.

### Phase 3: Manual / Exploratory Validation

Catches bugs requiring subjective judgment (comprehensibility of messages, appropriateness of error handling, edge cases tests didn't cover).

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
- [ ] Resource stability verified (soak test passed, no leaks)
- [ ] API contracts verified (no schema drift between components)
- [ ] Fault tolerance tested (non-critical dependency failure degrades gracefully)
- [ ] Error handler coverage (every catch block tested)
- [ ] No retry amplification (retry budget bounded under failure injection)

---

## Part 2: Bug Root Cause Investigation

> **Extracted for progressive disclosure.** Load **`phase-6-root-cause-investigation.md`** when a sub-phase fails. Contains: Layer Isolation (Q1–Q4), Evidence Collection (Q5–Q7), 5-Why Drill, Fix Verification (V1–V5), and 12 common bug patterns.

---

## Part 3: Integration Gap Detection Checklist

For each pair of interacting components, walk through every row. If any cell is "we don't know," that's your gap.

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

Phase 6 is the pipeline's **validation closure** — it validates Phases 1–5 output through systematic testing. It does NOT use Ralph loop. No reviewer subagent for go/no-go — the release decision is the user's business decision, not a technical review.

### Quality Mechanisms

| Mechanism | When | Purpose |
|-----------|------|---------|
| **Sub-phase gates** | Phase 0–3 (0 → 1 → 1.5? → 1.6? → 2 → 3) | Objective pass/fail verification |
| **追问 protocol** | Sub-phase failure | Root cause investigation with termination guarantees |
| **User go/no-go** | All sub-phases pass | Final business decision |

### Sub-phase Execution Rules

1. **Strict sequential**: Phase 0 → 1 → 1.5 (if applicable) → 1.6 (if applicable) → 2 → 3. No skipping.
2. **Gate is pass/fail**: No severity classification. Green or red.
3. **Any failure** → load `phase-6-root-cause-investigation.md`, run 追问 protocol → determine rollback target.
4. **All pass** → fill Release Gate Checklist with evidence → submit to user.

### 追问 Protocol Summary

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

| Root Cause Layer | Rollback To | Rerun Scope |
|-----------------|-------------|-------------|
| Test gap | Phase 4 | Phase 4 → 5 → 6 |
| Code bug | Phase 5 | Phase 5 → 6 |
| Design flaw | Phase 2 | Phase 2 → 3 → 4 → 5 → 6 |
| Requirement error | Phase 1 | Full pipeline |
| Config/environment only | Fix config | Phase 6 only (full rerun) |

**Rules:** (1) Phase 6 rerun is always full from Phase 0, never incremental. (2) Rollback to Phase 1–3 preserves existing code but requires new Ralph loops. (3) Config-only fixes need no code change.

### User Go/No-Go

When all sub-phases pass, submit the Release Gate Checklist with evidence. The user decides — go/no-go involves business factors (release window, user impact) beyond technical scope. The checklist provides evidence; the user adds judgment.
