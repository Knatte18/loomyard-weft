---
round: 1
ledger:
  - key: reconcile-walk-passes-merge-commit-report-head
    rounds: [1]
    status: resolved
  - key: errconflictswithrecord-signature-and-test-callers
    rounds: [1]
    status: resolved
  - key: recoverresult-warnings-comment-stale
    rounds: [1]
    status: resolved
  - key: record-batch-package-doc-lands-late
    rounds: [1]
    status: resolved
  - key: runinflight-unreadable-outcome-untested
    rounds: [1]
    status: resolved
  - key: ly-drive-recipe-blind-omits-batten
    rounds: [1]
    status: resolved
  - key: merge-in-test-seeds-primeworktree-not-anchorpath
    rounds: [1]
    status: resolved
  - key: card4-recorddeps-seams-underspecified
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so there was no prior ledger.
The reviewer approved with eight findings: one MEDIUM, four LOW and three NIT.
The fixer resolved all eight in the cards, and the judge checked each fix in place.

The one substantive finding was card 1's first-parent walk.
It checked the parent count before comparing with the report head, so it walked past a report head that was itself a merge commit.
Card 1 now compares first and tests that case.

The remaining fixes are minor:

- Card 2 now carries the doc half.
- Card 3 now edits `RecoverResult`.
- Card 5 now tests an unreadable `outcome.yaml`.
- Card 6 pins the signatures and the anchor seed.
- Card 4 names every seam it wires.

No finding is open.
