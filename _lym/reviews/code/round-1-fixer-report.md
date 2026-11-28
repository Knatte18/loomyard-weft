---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer report, round 1

## Changes

- **F1** (commit `4fe301c9c`): added `TestFinalize_HeadReadFailureFailsAWiredLanding` to `internal/landingshed/finalize_test.go`.
  A failed head read after the parent-side merge must fail `Call` with the injected error and leave `LandLocally` uncalled.
- **F2** (commit `b9fe8958d`): `recordHeldLanding` reads the parent branch before the pair push lock and the status commit.
  It passes the branch to `writeLedgerLanding(parent, branch, slug, tip, records)`.
  The test call and `_lym/landing/summary.md` follow the new signature.
- **F3** (commit `b9fe8958d`): narrowed the landing-commit wording in `internal/landingshed/deps.go` and in the three `finalize.go` comments.
  The proposed sentences are split to keep each line under the length limit.
  The same wording stood in the `landLocally` and `writeLedgerLanding` doc comments and in `_lym/landing/summary.md`, so those are narrowed too.
  No code changed for this finding.
- **F4** (commit `b9fe8958d`): the test records all three runs in one loop and drops the unrecorded-commit assertion.
  The commit carries a `Loosened-Check` trailer for it, and the summary's test paragraph follows.

F2, F3 and F4 share changed files, so they are one commit.

- **Comment lines** (commit `cebafaf0c`): the gate found an over-long line in the `pushParent` doc, added for F3.
  It also found a wrapped sentence in the new F1 test's doc.
  Both are rewritten as shorter sentences, one per line, with no code change.

Gate: `lym gate test --tier 0 --tier 1 ./internal/landingshed ./internal/loomcli` passed both tiers before the comment repair.
The new test ran in tier0, and the ledger landing tests ran in tier1.
After the repair, `lym gate test --tier 0 ./internal/landingshed` passed with no failures.
The new test was not run with the `headErr` check removed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, the `Deps.RecordLanding` doc "records commit as the run's landing commit": disputed, premise false.
  The next line of the same doc, `internal/landingshed/deps.go:261`, defines that commit as the parent pair's head right after a parent-side merge that landed.
  That holds on a re-run too, so the doc claims nothing the code does not pass.
- fix-proposal: no findings.
- fix-affected-set: no findings.
  It noted the same `Deps.RecordLanding` line and the `recordLandedSize` doc as out of scope.

## Disputed

None of the review's findings.

## Deferred

None.

## Rulings

None.
