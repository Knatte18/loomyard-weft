# Write a push-ledger landing from a given slug, tip and records commit

The push-ledger landing write took its values from the calling run,
and used the parent's head, read when the landing was recorded, as the tip.
A writer that lands several runs at once needs to record each run with that run's own landing commit as its tip.
A parent ahead of origin by such a chain of landing commits then reads as ahead by owed tips alone.

## What changes

- `internal/loomcli`: a new writer records one landing from values given as arguments:

  ```go
  func writeLedgerLanding(parent runpusher.Target, branch, slug, tip, records string) error
  ```

  `parent` is the parent pair's pusher target and `branch` the parent branch.
  `slug` is the landed run, `tip` its landing commit and `records` the records commit it holds back from origin.
  A standing landing of the run is replaced.
  `recordHeldLanding` still holds the pair's push lock, commits the run's status records
  and picks the records commit through `oldestHeldRecords`, and now writes the landing through `writeLedgerLanding`.
- `internal/landingshed`: Finalize reads the parent's head once, right after its parent-side merge,
  and hands it to both `Deps.RecordLanding` and `Deps.LandLocally`.
  The seam a Finalize reaches the writer through keeps its signature:

  ```go
  LandLocally func(tip string) error
  ```

  On a pass whose merge commits, that head is the run's own landing commit, so a lone run records the same commit as before.
  On a re-run over a parent that already holds the run's work, it is the parent's head.
  A failed head read is still only logged when no `LandLocally` is wired or the push is skipped,
  and still fails the call when `LandLocally` is reached.

## Design decisions

- `writeLedgerLanding` is told the parent target and branch rather than deriving them from a run's worktree,
  so it can record a run other than the caller's.
  No caller does so yet: a Finalize still writes only its own run's landing.
- A run recorded again through `landLocally` with its own landing commit keeps that tip.
  It also keeps its earlier records commit while origin lacks it and the new records commit descends from it,
  which is the existing `oldestHeldRecords` rule.

## Tests

- `TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed` lands three runs on the prime as a chain of landing commits
  and records each with its own tip and records commit.
  The parent reads as ahead by owed tips alone, both then and after run one is recorded again through `landLocally`,
  and every landing keeps its own tip and records commit.
- `TestFinalize_HeadReadFailureFailsAWiredLanding` pins that a failed post-merge head read fails a wired landing
  and records none.

## To check by hand

- `TestFinalize_HeadReadFailureFailsAWiredLanding` in `internal/landingshed/finalize_test.go` was added
  to cover the `headErr` branch of `pushParent` in `finalize.go`, which was untested earlier in this change.
  Check that it covers that branch.

## Merge danger

**Door:** two-way
**Blast radius:** landing
