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
  - id: F5
    disposition: ruling
    reason: "Carried out the parent's ruling (restore the old join results within the bound) except for the checkpoints of a pair with no archive tag that lie before the checkpoint walk's start, which are left undone."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

Three commits: `a428fb52f` holds every finding, because their changed files chain into one group (`batten.go`, `boardhistory.go`, `landings.go`, `runs.go`, `landings_test.go`, `runs_test.go`, `doc.go`), `d1584e6fa` holds the two doc fixes the self-check found, and a third repairs the gate's comment-format findings.
That third commit breaks each over-long or mid-sentence-wrapped comment the round added, in `batten.go`, `boardhistory.go`, `doc.go`, `landings.go` and `runs_test.go`, into one sentence per line.
It changes no code.
`lym gate test ./internal/reportengine` passed after the first two: 181 targets passed, 0 failed.

- F1, F2: `TestLandingJoiner_HoldsNoParsedBoard` carries a `//lym:seam` line for each in-module struct it still walks and a `//testtiming:keep` line.
  It follows the review's second option: `Sources`, `Options` and `Report` are the report's own inputs and outputs, so the walk skips them, and the marks list `landingJoiner`, `landingClaim`, `boardHistory`, `commitSpan`, `labelSpan`, `gitrepo.LogCommit` and `boardengine.Task`.
- F3: `snapshotsOfRun` takes the run's seed stamp and stops widening at the slug's earlier tag, or at `battenSeedMargin` before the stamp when there is no earlier tag.
  `widen` takes that bound, caps the doubled floor at it, and falls back to the whole history only with no bound.
  Rows in `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan` pin both stops.
- F4: the batten walk and the board walk follow the first-parent line, which stops at the first commit older than the floor.
  The test asserts that every board walk with a `Since` is first-parent, and the batten test asserts the same for every records walk.
- F6, F9: the batten test now asserts the `Since` of every records walk as an exact list, with the F9 row.
- F7, F8: `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn` is table-driven.
  The reviewer's F7 example puts the parent commit before the parent walk's start, so it cannot be a landing.
  I built the row differently: a done mark between the checkpoint and the floor closes the window before the landing, and the landing stays unmapped, whereas without the floor-lowering loop it would map.
  The F8 row is a quiet board read at its tip alone.
- F10: the board in effect at the floor is the first commit older than the floor on the first-parent line.
- F11: `doc.go` says the first walk opens a week before the parent walk's start, which lies `FreshDays` before the window for the convergence report and at the window for the timing report.
- F5, the ruling: see Rulings.

## Disputed

None.

## Deferred

None of the findings is deferred.
Two self-check findings are kept as they are and recorded in the Self-check section for the parent to rule on.

## Self-check

Forks: fix-code-correctness, fix-consistency, fix-proposal and fix-affected-set all returned reports, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, LOW: a batten seed committed more than a week before the loom seed stamp is no longer found.
  Merged with fix-proposal's MEDIUM on the same bound.
  Disposition: deferred to the parent, a real tradeoff.
  Any lower bound is a guess, and no bound brings back the whole-history walk of F3.
  The bound is the review's own proposal and doc.go states it.
- fix-code-correctness, LOW, and fix-proposal, MEDIUM: the first-parent records walks miss status and checkpoint commits that reach the main line only through a merge's second parent.
  Disposition: deferred to the parent, a real tradeoff.
  F4's `fix` key allows the first-parent walk only if the line is written linearly, and I found no evidence either way.
  The alternative is a new `gitrepo.LogQuery` option that ends a committer-time walk at `Since`, with integration tests under `internal/gitrepo`, which is outside the findings' files.
- fix-consistency, LOW: the doc said the records walk is "once" while it widens and the checkpoints have their own walk.
  Disposition: fixed in `d1584e6fa`.
- fix-consistency, LOW: the doc did not state the checkpoint walk's reach for a pair with no archive tag.
  Disposition: fixed in `d1584e6fa`.
- fix-affected-set: no findings.

## Rulings

- F5, parent ruling "F5: bounded reads keep the old join results": restore the old join results within the bound.
  Carried out:
  - The landing checkpoints come from `readCheckpoints`, a records walk of its own that no run's seed changes, so widening for another run no longer changes them.
  - The walk opens at the parent walk's start or the opening of the oldest pair a tag found no task commit for, whichever is older, and at the whole history when such a pair's opening was not reached.
    `taskCommitBelow` now reports that opening's time.
  - `readOlderLabels` reads board commits older than the window, newest first and once each, for each claimed slug that the board in effect lacks, until its entry is found or the history ends, and keeps only the labels.
  - Rows in `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn` pin the checkpoint reach and the older labels.
  Left undone: the checkpoints of a pair with no archive tag that lie before the checkpoint walk's start.
  Finding such a pair's opening needs an unbounded records walk, which the Goal removes, so that case still opens no time-join window.
  doc.go states it.
  The ruling is neither mistaken nor redundant, so I did not flag the parent separately.
