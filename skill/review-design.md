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

Between Recall and Precision passes, the main agent gathers these facts from the project:

```text
facts_to_gather = [
  read the relevant prior-phase outputs (to verify cross-phase consistency claims)
  check the requirements document (to verify completeness claims)
  read the project's coding conventions / RULES.md (to verify best-practice claims)
]
```

Then inject gathered facts into the Precision Filter prompt from `review-precision-filter.md`.
