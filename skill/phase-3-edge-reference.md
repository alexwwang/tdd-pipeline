# Edge Case Depth Reference (Reviewer Only)

This file is loaded by the Ralph reviewer during Phase 3 gate review.
Do NOT load this during deliverable creation — it must not influence the main agent's independent edge case identification.

The reviewer checks each assertion below. Where the deliverable's coverage is thinner than the assertion requires, issue an M-level severity with a constructive suggestion referencing the specific gap.

---

## Serialization Boundary

Assert the deliverable covers:
- Numeric precision drift over repeated operations (not just single-operation accuracy)
- Encoding/decoding round-trip across system boundaries (including mixed-encoding scenarios)
- Special characters that change meaning across contexts (null vs empty vs missing, Unicode beyond BMP)
- Date/time format consistency across components (timezone, precision, format variations)

## Error Handler Correctness

Assert the deliverable covers:
- Every catch/except block has ≥1 test triggering it
- Error context preserved through the error chain (cause, stack trace, error codes, original input)
- Cleanup/finally runs even on error — no resource leaks on failure paths
- System is recoverable after error (state remains valid, retry works)

## Implicit Contract

Assert the deliverable covers:
- Call order dependencies verified (A must complete before B, including across components)
- Sync vs async error propagation tested separately
- Shared state thread safety under concurrent access (if applicable or parallelizable)
- Contract assumptions between components explicitly tested (e.g., data shape, field order)

## Resource Stability

Assert the deliverable covers:
- N-iteration soak test for resource-heavy operations (memory, FDs, connections measured before/after)
- Resource cleanup on all error paths (not just happy path)
- Unbounded accumulation scenarios (error lists, log buffers, cache growth)

## Cascading Failure Prevention

Assert the deliverable covers:
- Non-critical dependency failure doesn't propagate to core flow (failure isolation)
- Retry budget prevents amplification (retry storms, exponential backoff correctness)
- Timeout propagation: caller timeout ≤ callee timeout (nested call chains)
- Partial failure handling: some channels/components fail while others succeed

## Performance Logic

Assert the deliverable covers:
- Hot paths tested with realistic data volume (not just 1-item trivial cases)
- N+1 detection: batch operations verified not to trigger per-item overhead
- Slow paths remain within latency budget under expected load
- Large-input memory behavior (streaming vs buffering, OOM risk)
