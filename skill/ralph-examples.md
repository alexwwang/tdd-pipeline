# Ralph Loop — Concrete Examples

Loaded on demand when encountering contested issues or ambiguous stop/gate scenarios. See `ralph-review-loop.md` for the core protocol.

## Stop Condition Examples

**Example A — Cannot stop (intermittent zeros):**
```
Round 1: H=1, M=2 → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 new findings → consecutive counter = 1 → continue (NOT stop — need 2 consecutive!)
Round 4: M=1      → Fix → counter RESET to 0 → continue
Round 5: 0 new findings → counter = 1 → continue
Round 6: 0 new findings → counter = 2 → ✅ STOP (rounds 5 & 6 both 0 new findings)
```

**Example B — Cannot stop (only 1 zero + L resets counter):**
```
Round 1: H=1      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 new findings → counter = 1 → continue (NOT stop — need 2 consecutive!)
Round 4: L=1 (new) → Counter RESET to 0 (any new non-I finding resets; fixing L is optional) → continue
Round 5: 0 new findings → counter = 1 → continue
Round 6: 0 new findings → counter = 2 → ✅ STOP
```

**Example C — Correct stop (at round 4):**
```
Round 1: M=2      → Fix → continue
Round 2: M=1      → Fix → continue
Round 3: 0 new findings → counter = 1 → continue
Round 4: 0 new findings → counter = 2 → ✅ STOP (rounds 3 & 4 both 0 new findings, consecutive)
```

**Example D — Persistent issues escalation (model-driven, no round cap):**
When the same C/H/M1 finding recurs across ≥ 3 rounds despite fix attempts, the model must assess whether this is a prior-phase root cause (unclear requirement, design flaw, test gap) rather than an implementation issue. If so, either escalate to the user for clarification or rollback to the prior phase.

## Contested Issue Examples

**Example E — Contested issue lifecycle (reviewer re-raises, then drops):**
```
Round 3: reviewer finds [M-3] Missing timeout handling in API client
  → main agent REJECTs: "Timeout is handled by the gateway, not the client (see Phase 2 §Component Gateway, line 42)"
  → [M-3] becomes contested, forwarded to Round 4
  → Tally still includes [M-3] in M count

Round 4: reviewer receives contested [M-3] in context
  → reviewer checks Phase 2 Gateway component → confirms timeout IS handled there
  → reviewer ACCEPTS rejection → [M-3] dropped from tally
  → No new C/H/M1 issues → tally = 0
  → Consecutive-zero counter = 1 → continue
```
> **Key lesson**: The reviewer is not obligated to fight every rejection. When the main agent
> provides specific, verifiable evidence (line reference, component specification), the reviewer
> should check that evidence and accept the rejection if it holds. This is the "graceful
> concession" pattern — it keeps the review loop moving and builds trust for future disputes.

**Example F — Contested issue resolved via MODIFY (real: ky prefixUrl dispute [sindresorhus/ky]):**
```
Round N: reviewer finds [H-2] prefixUrl rejects leading slashes, blocking standard API workflows
  (GitHub, Reddit, Twilio, Netlify, Twitter/X, Salesforce docs all use /path format)
  → main agent REJECTs: "This is documented intentional behavior. prefixUrl does string
     concatenation, not URL resolution. Leading slashes create false expectations."
  → [H-2] becomes contested, forwarded to Round N+1
  → Tally still includes [H-2] in H count

Round N+1: reviewer investigates evidence
  → reviewer finds: 6 major API docs ALL use leading-slash paths
  → reviewer re-raises [H-2]: "Documented intent is acknowledged, but user-side cost is
     library incompatibility with standard practice. The design principle is valid;
     the strict rejection of /path is the problem."
  → main agent cannot REJECT on the same grounds ("documented intentional behavior")
  → main agent MODIFYs: "Add slash stripping as a normalization step before concatenation.
     Preserves prefixUrl's string-concatenation semantics while accepting standard API paths."
  → Rationale: Keep the design principle (prefixUrl = concatenation, not URL resolution),
     adjust the implementation (strip leading slash instead of rejecting it).
  → [H-2] still in H tally (contested, pending verification)

Round N+2: reviewer verifies the slash-stripping modification
  → Modification correctly implements the normalization
  → Zero new issues → tally = 0
```
> **Key lesson**: "Documented intentional behavior" is a valid first-round REJECT, but when the
> reviewer produces evidence that the intentional behavior creates widespread user harm, the
> same rationale cannot be reused. MODIFY preserves the design principle while addressing the
> practical cost — this is the canonical "principled compromise" pattern. ADOPT = reviewer's
> evidence fully invalidates the original rejection (issue was correct as-stated).
> MODIFY = evidence warrants action but the design principle should be preserved.

**Example G — Contested issue escalation to user with dossier (real: requests URL parser deadlock [psf/requests#6927]):**
```
Round N: reviewer finds [H-1] URL parsing regression breaks IPv6 zone ID support
  (urlparse cannot handle [fe80::1%eth0], causes socket errors on multi-NIC machines)
  → main agent REJECTs: "Using stdlib urlparse is correct. urllib3.parse_url has its own
     compatibility issues. Every URL parsing change is a minefield."
  → [H-1] contested, forwarded to Round N+1

Round N+1: reviewer provides counter-evidence
  → reviewer: "stdlib urlparse explicitly does NOT support RFC 6874 zone identifiers.
     Socket errors confirmed on multi-NIC machines. User reports filed."
  → reviewer re-raises [H-1] with RFC 6874 citations and concrete error traces
  → main agent REJECTs again on NEW grounds (2nd dispute round):
     "Historical evidence shows every parser replacement causes regressions.
      Risk outweighs the fix."
  → ⚠️ Rule 3 doesn't apply — grounds shifted ("stdlib correctness" → "regression risk"),
     constituting a new rejection rationale. Rule 4 still triggers at 2-round limit → ESCALATE
  → [H-1] suspended from gate tally (pending external resolution)

=== Escalation Dossier ===
1. Original Issue: urlparse regression in v2.32.3 breaks IPv6 zone ID URLs
   Severity: H — causes socket errors on multi-NIC production machines
2. Reviewer Evidence Chain:
   - RFC 6874: zone identifiers are standards-track, not edge cases
   - Concrete socket.gaierror traces on [fe80::1%eth0]
   - Affected user reports (link to issues)
3. Main Agent Rejection Rationale:
   - Round 1: stdlib urlparse is correct; urllib3.parse_url has compatibility issues
   - Round 2: every prior parser replacement caused regressions (historical record)
4. Dispute Trajectory:
   - Round N: reviewer raises [H-1], main agent REJECTs (stdlib correctness)
   - Round N+1: reviewer provides RFC + error evidence, main agent REJECTs (regression risk)
5. Current Status: regression is live, affecting multi-NIC users
=== End Dossier ===

User decision: "Conditional accept — use urllib3.parse_url ONLY for URLs containing
zone IDs, keep urlparse for all others." (This matches the eventual PR #7065 approach.)
```
> **Key lesson**: The escalation dossier is not "both sides argued, you decide." It is a
> structured evidence package: (1) what was found, (2) reviewer's evidence chain, (3) main
> agent's rejection rationale per round, (4) dispute trajectory, (5) current impact. The user
> should be able to make an informed decision without re-reading the entire review log.
