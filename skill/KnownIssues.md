# Known Issues — Phase 8 Acceptance Testing

Last updated: R20 (second dual-pass review gate passed R19+R20)

## P-tier (Proposals / Improvements)

### KI-01 | P | L161-162 | Merge informational Phase 7 findings categories
- **Raised-in**: R18
- **Re-raised-in**: R19, R20
- **Description**: Phase 7 findings Category 4 (unresolved P) and Category 5 (rejected findings) have identical handling (informational, no risk assessment). Could merge into a single "Informational findings" category.
- **Why deferred**: Cosmetic simplification; both categories function correctly as-is.
- **Plan**: Consider merging in next edit pass.

### KI-02 | P | L21 | Unify zero-AC + Category 3 abbreviated report path
- **Raised-in**: R19
- **Re-raised-in**: R20
- **Description**: Zero-AC edge case and Category 3 (C/H findings) have overlapping but independently specified abbreviated report handling. Could unify into a single "abbreviated report" path invoked by both.
- **Why deferred**: Both paths produce correct output; unification is structural improvement only.
- **Plan**: Add a shared "Abbreviated Report" subsection referenced by both edge cases.

### KI-03 | P | L25 | Replace "Rollback Guidance" reference with "C/H classification reference"
- **Raised-in**: R16, R17, R19, R20
- **Description**: L25 references Phase 7's "Rollback Guidance section" as a load dependency. Phase 8 only needs C/H classification definitions, not rollback targets. Decoupling would strengthen scope boundary.
- **Why deferred**: Loading the section ≠ executing it. Multiple Precision rounds (R16, R17, R18, R19, R20) all REJECTED the scope violation claim. However, the wording could be tighter.
- **Plan**: Reframe as "Phase 8 references phase-7-system-quality-audit.md for Phase 7 findings format and severity definitions."

### KI-04 | P | L203 | Secondary AC release-ready formula not explicit
- **Raised-in**: R17
- **Description**: Coverage Summary formula "Secondary ACs: X out of Y release-ready, W accepted manual-only (Z warnings)" has overlapping sets (✱ accepted Weak are both release-ready and manual-only). The relationship X + Z ≠ Y is never made explicit.
- **Why deferred**: Readers can derive the formula; explicit statement is nice-to-have clarity.
- **Plan**: Add a parenthetical note after L203 clarifying the set relationships.

### KI-05 | P | L286 | Part 4 correction cycles — no cascade escalation
- **Raised-in**: R19, R20
- **Description**: Maximum 3 correction-verification cycles; if cascading corrections exceed 3, "escalate to user for manual review." No specific guidance for cascading threshold-dependent corrections.
- **Why deferred**: Escalation to user covers the case; the existing rule is sufficient.
- **Plan**: No action unless cascade issues surface in practice.

## L-tier (Cosmetic / Readability)

### KI-06 | L | L25 | Execution paragraph too long
- **Raised-in**: R18, R19, R20
- **Description**: The Execution block is a single ~15-line paragraph covering edge cases, load dependencies, merge rules, AC renumbering, conflict resolution, and supplementary data. Should be broken into sub-sections.
- **Why deferred**: Structural improvement only; no logical or scope defect.
- **Plan**: Break into titled sub-paragraphs in next edit pass.

### KI-07 | L | L80 | Status symbols paragraph dense (~200 words)
- **Raised-in**: R17, R18, R19, R20
- **Description**: The status symbols note packs 5 symbol meanings, 2 sub-cases, cross-references, and caveats into a single dense paragraph. Would benefit from a table format.
- **Why deferred**: Reference paragraph functioning correctly; table format is readability improvement.
- **Plan**: Convert to table in next edit pass.

### KI-08 | L | L156-162 | Phase 7 findings table mixed row types
- **Raised-in**: R19, R20
- **Description**: Categories 1–5 mix operational actions and risk assessment in a format that's hard to scan. L219 Note explains the mixed types but the table itself lacks visual grouping.
- **Why deferred**: Table is functional; Note provides the necessary context.
- **Plan**: Add grouping separators in next edit pass.

### KI-09 | L | L80 vs L113 | Bad Requirement dual symbol (❓→❌)
- **Raised-in**: R20
- **Description**: Diagnosis Table uses ❓ for Bad Requirement; the matrix Status column maps it to ❌. Dual-symbol system adds cognitive load.
- **Why deferred**: The mapping is documented at L80. No functional impact.
- **Plan**: Consider consolidating to ❌ with a note in next edit pass.

## I-tier (Informational — no action)

### KI-10 | I | L38 | Phase 6 re-run metadata contract wording
- **Raised-in**: R19, R20
- **Description**: L38 uses "must include" when specifying Phase 6 re-run metadata. Technically prescribes upstream format. Precision consensus: this is an interface contract (Phase 8's input requirement), not a quality standard. Wording could be softened.
- **Why deferred**: Consensus established across R16–R20 that diagnostic input requirements ≠ upstream quality standards.
- **Plan**: Consider softening to "Phase 8 checks for..." phrasing.

### KI-11 | I | L122 | Escalated Unknown re-invocation model implicit
- **Raised-in**: R18, R19, R20
- **Description**: Escalated Unknown classification implies Phase 8 re-invocation after user-directed Phase 6 re-run. This lifecycle is never narrated explicitly (R19 added a note, but the full re-invocation flow remains implicit).
- **Why deferred**: Part 5 Step 1 serves as the re-invocation path; logic is sound.
- **Plan**: No action needed; R19 note at L122 is sufficient.

### KI-12 | I | L306 | Post-verification report modification without re-verification
- **Raised-in**: R17, R18, R19, R20
- **Description**: User accepting escalated Unknown modifies the Part 4-verified report without re-verification. Document explicitly states "this is intentional." Rejected as defect in 4 consecutive Precision rounds.
- **Why deferred**: Deliberate design decision. Recording user decision ≠ making independent judgment.
- **Plan**: No action.

---

## Summary

| Tier | Count | Action required |
|------|-------|----------------|
| P | 5 | Open — consider in next edit pass |
| L | 4 | Open — cosmetic improvements |
| I | 3 | Closed — informational only, no action |
| **Total** | **12** | **0 blocking** |

## Final KI Evaluation (per ralph-continuation.md §Final Evaluation)

- **Triage accuracy**: ACCURATE. All entries correctly classified; no C/H/M leakage into P/L.
- **Stale entries**: None. All entries raised in R16–R20; none have been fixed since raising.
- **Carry-forward**: None required for next phase. These are Phase 8 document quality improvements only.
