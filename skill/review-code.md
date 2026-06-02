# Code Review (Phase 4–5)

This file is loaded ONLY during code reviews (Phases 4–5). For design reviews, load
`review-design.md` instead.

## Code Review Checklist

The reviewer must verify each item below. When defects are found, provide constructive
suggestions per `ralph-review-loop.md` §Reviewer Output Requirements. When substantive
strategic concerns exist, provide critical opinions.

- [ ] **Test quality**: Completeness, edge cases, descriptive names, one assertion per test where practical
- [ ] **Code quality**: Clean code, no duplication, proper abstractions, follows language idioms
- [ ] **TDD compliance**: No untested code; no code exists without a failing test justifying it
- [ ] **Correctness**: Logic errors, missing cases, broken invariants
- [ ] **Completeness**: Dead code remnants, missed cleanups, orphaned references (imports/columns/variables)
- [ ] **Consistency**: Naming inconsistencies, mixed conventions, partial refactoring
- [ ] **Security**: Data exposure, injection risks, missing validation
- [ ] **Performance**: Unnecessary computation, resource leaks, N+1 patterns
- [ ] **Maintainability**: Readability, documentation gaps, confusing abstractions
- [ ] **Backward compatibility**: Public API removal/signature changes safe?
- [ ] **Import cleanup**: No residual imports/exports of deleted symbols

### Phase 4 Specifics

- Are ALL tests genuinely failing? No premature implementation?
- Do tests import non-existent modules/functions as intended?
- Are stubs truly minimal and not passing tests?

### Phase 5 Specifics

- Is refactoring clean? Any regressions introduced?
- Is the minimum code principle respected? No gold-plating?
- Does every line of business code have test coverage?
- Are review prompt contents clean? (No round counts, tallies, or fix lists leaked
  into reviewer prompts — RPS will flag these)

## Single-Pass Review

Use the standard reviewer prompt from `ralph-review-loop.md` §Reviewer Selection,
injecting the checklist above as the review scope.

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

1.  Correctness: Logic errors, missing cases, broken invariants.

2.  Completeness: Dead code remnants, missed cleanups, orphaned references
    (imports, column names, variables).

3.  Consistency: Naming inconsistencies, mixed conventions, partial refactoring.

4.  Security: Data exposure, injection risks, missing validation.

5.  Performance: Unnecessary computation, resource leaks, N+1 patterns.

6.  Test quality: Missing assertions, weakened coverage, tests that should not
    have been deleted.

7.  Maintainability: Readability, documentation gaps, confusing abstractions.

8.  TDD compliance:
    - Phase 4: Are all tests genuinely failing? No premature implementation?
    - Phase 5: Is the minimum implementation principle respected? No over-engineering?

9.  Backward compatibility: Are public API removals or signature changes safe?

10. Import cleanup: Any residual imports/exports of deleted symbols.

11. Security deep review: Trace all external input data flows from entry point to
    use site — are column names/values used in string-concatenated SQL? Is
    validation enforced at every trust boundary? Are there bypass paths? Is
    sensitive data leaked in logs or error messages?

12. Test gap review: Does every AC have a corresponding test? Are edge cases
    covered? Are error paths tested? Are there tests that should exist but don't?
    Are there non-deterministic assertions? Check all ORDER BY/LIMIT queries: when
    multiple rows share the same sort key, is the result non-deterministic? Is there
    a unique tiebreaker?

13. Resource safety: Are opened resources (files, connections, handles) correctly
    closed on all paths including error paths? Do any resources accumulate in loops
    or recursion? Are there connection pool exhaustion scenarios? Check every WHERE
    clause and JOIN condition column for index coverage (note composite PK prefix
    scan limitation: PK(a,b) covers WHERE a=? but NOT WHERE b=?).

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
  "location": "{file:line or description}",
  "description": "{issue description}",
  "evidence": "{specific evidence reference}",
  "suggestion": "{fix suggestion}",
  "confidence": 0.0-1.0
}
```

## Fact-Gathering for Precision Filter (Code Review)

Between Recall and Precision passes, the main agent gathers these facts from the
codebase:

```text
facts_to_gather = [
  grep for residual references to deleted symbols (classes, functions, variables, columns)
  cat the actual implementation of suspect functions (not the diff, the full code)
  read config files referenced in findings (to verify staleness)
  check __all__ / exports for orphaned references
]
```

Then inject gathered facts into the Precision Filter prompt from
`review-precision-filter.md`.
