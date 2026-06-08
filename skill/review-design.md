# Design Review (Phase 1–3)

This file is loaded ONLY during design reviews (Phases 1–3). For code reviews, load `review-code.md` instead.

## Design Review Checklist

The reviewer must verify each item below. When defects are found, provide constructive suggestions per `ralph-review-loop.md` §Reviewer Output Requirements. When substantive strategic concerns exist, provide critical opinions.

- [ ] **Axiom 1 — Independence** (Phase 2): FR-DP uncoupled (or decoupled with adjustment order documented)
- [ ] **Axiom 2 — Information** (Phase 2, if alternatives): Coupled designs eliminated, simplest remaining selected
- [ ] **FR Purity**: ACs describe WHAT not HOW (no implementation leakage)
- [ ] **Completeness & Traceability**: All prior-phase requirements covered; every AC maps to a test or design element
- [ ] **Consistency**: No contradictions with prior phase outputs
- [ ] **Clarity**: Unambiguous, explicit, no hand-waving
- [ ] **Correctness & Boundaries**: Every AC has testable verification; edge cases, error paths, and failure modes covered; no non-deterministic behavior
- [ ] **Feasibility**: Can this actually be built and tested?
- [ ] **Backward compatibility**: API changes won't break callers
- [ ] **Security deep review**: Track all external input data flows — trust boundaries, per-entry-point auth/validation, bypass paths, sensitive data exposure
- [ ] **Resource/Performance**: Scale behavior; resource cleanup on all paths (including errors); index coverage for WHERE/JOIN columns

## Single-Pass Review

Use the standard reviewer prompt from `ralph-review-loop.md` §Reviewer Selection, injecting the checklist above as the review scope.

## Dual-Pass Recall Prompt

If using dual-pass mode, inject this as the Recall subagent prompt:

```text
You are an independent review expert (Recall Pass). Your job is to find all potential
issues as comprehensively as possible. A second expert will filter your findings for
precision.

Core principle: over-report rather than miss.
- If unsure whether an issue is real, report it anyway with lower confidence.
- If something looks suspicious but lacks full evidence, report it.
- Do not skip anything questionable just because it "might not be a problem".

## Input Contract

This prompt contains the following injected variables. Do not modify them.

| Variable | Source | Format |
|----------|--------|--------|
| {REVIEW_SCOPE} | Current phase Gate checklist | Structured text (IN SCOPE / OUT OF SCOPE) |
| {DELIVERABLE} | Current phase output | Full document content |
| {PRIOR_PHASE_OUTPUTS} | Previous phase deliverables | Document list |
| {CONTESTED_ISSUES} | Prior round REJECTed C/H/M | JSON list (empty for Round 1) |
| {KNOWN_ISSUES} | KI document | Structured text (injected every 3 rounds) |

## Review Scope

Injected by the main agent from the current phase file. Do not modify.

{REVIEW_SCOPE}

⛔ OUT OF SCOPE items: if worth noting, use severity I with tag [DEFERRED].
   Must NOT be labeled C/H/M/P. Do not enter the fix loop.

## Review Dimensions

Check only the IN SCOPE dimensions declared in {REVIEW_SCOPE}:

1. ⭐ Independence Axiom (Phase 2 only): Is the FR-DP design matrix constructed?
   Is there coupling (one FR affected by multiple DPs, or one DP affecting multiple
   FRs)? Are quasi-coupled designs documented with adjustment order?

2. ⭐ Information Axiom (Phase 2 only, when alternatives exist): Have coupled designs
   been eliminated? Is the simplest remaining design selected (fewest coupling points)?

3. ⭐ FR Purity (Phase 1/2): Do ACs describe WHAT (functional domain) not HOW
   (physical domain)? Is implementation detail leaking in (e.g. specific technology
   choices)?

4. Completeness & Traceability: Are all prior-phase requirements covered? Does every
   AC map to a test or design element?

5. Consistency: Any contradictions with prior phase outputs?

6. Clarity: Unambiguous, explicit, no hand-waving?

7. Correctness & Boundaries: Does every AC have a testable verification condition?
   Are edge cases and error paths covered? Any non-deterministic behavior (e.g.
   sorting without tiebreaker)?

8. Feasibility: Can this actually be built and tested?

9. Backward Compatibility: Will API changes break callers?

10. Security Deep Review: Trace all external input data flows — which entry points
    accept external input? Where does data cross trust boundaries? Can sensitive data
    be exposed to unauthorized parties? Is auth/validation enforced at every entry
    point? Are there validation bypass paths?

11. Resource/Performance: How does behavior change as data grows? Are
    connections/handles correctly closed on all paths including error paths? Any
    potential memory growth or resource exhaustion? Check every WHERE/JOIN column for
    index coverage (note composite PK prefix scan limitation: PK(a,b) does not cover
    WHERE b=?).

## Deliverable

{DELIVERABLE}

## Prior Phase Outputs (for cross-phase consistency)

{PRIOR_PHASE_OUTPUTS}

## Contested Issues (from prior round)

{CONTESTED_ISSUES}

## Output Format

For each finding, output strictly as JSON:
{
  "id": "F-{sequence}",
  "severity": "C|H|M|P|L|I",
  "category": "{category}",
  "location": "{file:line or document section}",
  "description": "{issue description}",
  "evidence": "{specific evidence reference}",
  "suggestion": "{fix suggestion}",
  "confidence": 0.0-1.0
}
```

## Fact-Gathering for Precision Filter (Design Review)

Between Recall and Precision passes, the Fact-Gather subagent (see `review-fact-gather.md`)
locates relevant documents for each finding. The following investigation guide is injected
as `{FACT_INVESTIGATION_GUIDE}` into the Fact-Gather prompt:

```text
FACT_INVESTIGATION_GUIDE (Design Review — Phase 1–3):

For each finding, locate:
1. The relevant prior-phase outputs cited or implied by the finding
   - Cross-reference finding's claims against actual prior-phase deliverables
   - Path patterns: .tdd-pipeline/{yymmdd-summary}/p{N-1}-*.md
2. The requirements document sections relevant to the finding
   - Verify completeness claims against actual requirements
   - Path patterns: .tdd-pipeline/{yymmdd-summary}/p1-*.md
3. The project's coding conventions / RULES.md if referenced
   - Verify best-practice claims against actual conventions
   - Path patterns: RULES.md, docs/conventions.md, .editorconfig
4. The deliverable section directly cited in the finding's location field
   - The primary source for the finding's claim
```
