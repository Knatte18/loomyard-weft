---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

## Changes

- F1: `internal/landingshed/prreview_test.go`, `TestPRReview_Call`, subtest `OptionalSeamsLeftUnsetRunTheRow`, loops over a literal list of `ParentReviewWanted`, `AskParent`, `BeamRecord` and `DropBeamRecord`, with one `t.Run` per name that zeroes only that field and asserts `Call` returns no error.
  `optionalPullRequestSeams` and `checkWiring` are unchanged.
  Committed as `8e4fe534`.
- F2: `_lym/landing/summary.md:23`, the production comment endpoint changed from 2,574 to 2,576.
  The file is under `_lym`, so it is edited in the working tree and not committed.
  The test-only commit leaves the production comment count at 2,576.

`lym gate test ./internal/landingshed` passed: 842 targets, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
