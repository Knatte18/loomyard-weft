---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/landingshed/finalize_gate_test.go:708
    summary: The pending-findings resume test asserts only that Call returned no error, so it never checks that the wantLand row lands or that the other row does not
    fix: In the subtest at the "a resume with a fix review's findings pending" table, replace the bare Call and err check with the outcome-checking path. For the wantLand row call fx.call(t) and then fx.requireLanded(t, outcome, ptr). For the reviewStartErr row keep the Call error assertion and also assert len(fx.merger.calls) == 0, as the "fix review session that fails to start" subtest does.
---

### [LOW] Pending-findings resume test does not assert the landing

**Location:** `internal/landingshed/finalize_gate_test.go:708`

**Issue:** The `wantLand` row is named "the fixed change within the caps lands", but the subtest only compares `tt.wantLand != (err == nil)`.
A Stuck outcome also returns a nil error, so the row passes if Finalize halts after the fixer's fixed end instead of landing.
The sibling subtests assert `requireLanded` or `len(fx.merger.calls) == 0`; this one leaves the behavior its name claims unpinned.
This matters for the second focus question: the `gateAndFix` branch that runs the fixer on a passing gate and the clearing of `FixReview` after a fixed end are pinned, since the fixer-first order, the `fix_review_path` value and the cleared record are asserted. The landing that follows is not.

## Focus verdicts

- Acceptance bound to the HEAD: not a defect.
  `AcceptFixReview` stores `taskHead()` in `AcceptedHead`, and `judgeFixerChange` proceeds only when `rec.AcceptedHead == head`.
  A different HEAD resets `AcceptedHead`, `ReviewRounds` and `Findings`, so a fixer turn or operator commit after the accept reaches the cap check and the fix review.
  `gateAndFix` keeps the acceptance across the resume's `Halt` reset and clears the rounds only when none exists.
  The tests at `finalize_gate_test.go:618` and `:668` cover the same-HEAD, earlier-HEAD, no-acceptance and fixer-after-accept paths.
- The accept verb commits the change description to the weft through `CommitAnchoredPaths`, so the task HEAD read before the commit stays the HEAD the resume finds.

## Rejected

- A parent that moved between the accept and the resume makes `mergeParent` add a merge commit, which voids the acceptance and re-runs the fix reviews. This follows the same HEAD-binding that `Reviewed` uses and the decision record does not ask for more; not raised.
  Round 1's fixer disputed the same point with the ground that a looser rule would let a changed tree land on a stale acceptance, and I find that evidence holds.
