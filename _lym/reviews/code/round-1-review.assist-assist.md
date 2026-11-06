---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/planindex/planindex.go:29
    summary: The shared check-name migration leaves two readers comparing repeated literals
    fix: Import internal/planindex in internal/planglyph/deleteorder_test.go and internal/planglyph/planfixture_helper_test.go, and replace the f.Check comparisons against the delete-before-reference string at lines 240 and 106 respectively with planindex.CheckDeleteBeforeReference. Keep the existing assertions and helper behavior; no new test is needed.
---

### [BLOCKING] Complete the check-name migration

**Location:** `internal/planindex/planindex.go:29`; remaining readers at `internal/planglyph/deleteorder_test.go:240` and `internal/planglyph/planfixture_helper_test.go:106`.

**Issue:** The diff introduces a shared check-name constant and migrates every emitter and the dispatch classifier, but leaves the direct finding assertion and the shared site-filter helper comparing the old literal.
`PATTERN-code-conventions` requires a name that can go stale to be declared once and referenced everywhere; it does not exempt test readers.
These comparisons independently encode the name the new constant owns, so a later rename leaves the assertion stale and makes the helper discard the renamed findings.
This is an incomplete reader migration in the diff, not a request to change unrelated existing checks.
The round's constraints require a BLOCKING finding for a listed-rule violation.

## Review evidence

Reviewed `b3b736f7bf572b2541e5ab5580418f2ff14d8fef..HEAD` against the imported board entry in the Sketch, the focus directive, and the landing summary.

- `orderOnly` changes only for blocking findings, and the nonempty blocking-set guard prevents an empty or informational-only set from returning a conflict.
  Mixed blocking sets preserve the existing begun-batch distinction.
- `PlanConflictError.Error` preserves the rebaseline and numbered begin-batch tail while changing the ordering remedy.
  The CLI copies the cause into its output and conflict record without an enum check.
  `ReadConflictRecord`, `FixPlanSpawnOrAttach`, and `ClaimConflictRoute` route by findings rather than cause, so `plan_order` is accepted.
  The plan-fixer stencil sends changes beyond target respelling to the plan writer; the new classification does not require expanding that authority.
- All delete-order emit sites use the constant.
  Whole-tree searches found the remaining comparison readers reported in F1, plus the migrated engine classifier and its new test fixtures.
- Plan gates already call the delete-order analysis through `planGatePass`; dispatch calls `LaterDeleteReferences` over pending cards.
  Handle targets are excluded until bound, and the checks read current source.
  The choice to retain the existing checks satisfies the entry's request to consider earlier checking.
- The new table covers ordering-only sets, mixed sets in both begun states, and an informational ordering finding beside a different blocking finding.
  These classification cases are distinct from the existing `TestBeginBatch_ReResolvesPlanAtDispatch` cases for generic conflicts and informational-only dispatch.
  The added test contains no wait or sleep.

Validation was attempted with `timeout 3600s lym gate test ./internal/websterengine ./internal/planglyph ./internal/planindex ./internal/webstercli --tier 0`.
The gate could not register its runner because its lock under `/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/runproc/` was on a read-only filesystem; it reported no discovered or executed tests.
No test-pass claim is made, and this environment limitation is not filed as a code finding.
