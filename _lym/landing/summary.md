# Write a push-ledger landing from a given slug, tip and records commit

A push-ledger landing used to be written from the calling run's own worktree,
with the parent's head as its tip, read when the landing was recorded.
A writer that lands several runs at once needs to record each run with its own landing commit as the tip.
Otherwise a parent ahead of origin by a chain of landing commits holds commits no ledger landing names,
and it reads as ahead by more than owed tips.

## What changes

- `internal/loomcli`: a new writer records one landing from values given as arguments:

  ```go
  func writeLedgerLanding(parent runpusher.Target, slug, tip, records string) error
  ```

  `parent` is the parent pair's pusher target, whose current branch becomes the landing's `ParentBranch`.
  `slug` is the landed run, `tip` its own landing commit and `records` the records commit the landing holds back from origin.
  A standing landing of the run is replaced, as `runpusher.RecordLanding` already does.
  `recordHeldLanding` still holds the pair's push lock, commits the run's status records and picks the records commit through `oldestHeldRecords`.
  It now writes the landing through `writeLedgerLanding`.
- `internal/landingshed`: Finalize reads the parent's head once, right after its parent-side merge.
  That commit, the run's own landing commit, goes to both `Deps.RecordLanding` and `Deps.LandLocally`.
  The seam keeps its signature:

  ```go
  LandLocally func(tip string) error
  ```

  For a lone run the commit is the same one each of the two records read separately before.
  A failed head read is still only logged when no `LandLocally` is wired, and still fails the call when one is.

## Design decisions

- The writer takes the parent target, not a branch name, and reads the branch from the parent pair.
  A caller that already holds the target therefore needs no extra read to call it.
- A run recorded again through `landLocally` keeps its tip because the tip now comes from the caller.
  It keeps its earlier records commit while origin lacks it and the new records commit descends from it, which is the existing `oldestHeldRecords` rule.
- The writer runs nothing at its own initiative: a Finalize still writes only its own run's landing.
  No caller yet records another run.

## Tests

`TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed` lands three runs on the prime as a chain of merge commits.
It records each run with its own landing commit and records commit.
The parent reads as not ahead by owed tips alone while the last commit is unrecorded, and as ahead by owed tips alone once all three are recorded.
Run one is then recorded again through `landLocally` with its own landing commit.
Afterwards every landing still carries its run's own tip and records commit, and the parent still reads as ahead by owed tips alone.

## Merge danger

**Door:** two-way
**Blast radius:** landing
