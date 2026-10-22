---
verdict: APPROVED
---

## Scope against the board entry

The decision record has no `Added after Discussion` entries, so the board entry `done-check-spaced-create-paths` is the measure.
Its operator decision of 2026-10-22 keeps `webster validate` unchanged and moves the `loop-exited` halt reason to `loop-exit-halt-names-its-cause`.
The diff of pull request 840 at `a61be72c4` against `main` covers every remaining point:

- Issue #805: `DoneChecks` sends a key no glyph spells, such as `tree/with space/Spaced.cs`, to a check on disk instead of quarry (`internal/planglyph/donecheck.go`, `planparser.IsDiskCheckedTarget`).
  Tests cover spaced Create, Delete and Rename paths, both present and absent, and a bare `LICENSE`.
- Issue #809, points 2 and 4: a batch failed on a `glyph-rejected` finding now names amend, rebaseline and recover-batch as its way forward, or the fresh restart when the record lists uncheckable entries (`glyphRejectedWayForward`).
  Record-batch and recover-batch both take this path through `failCardNotDone`.
  The disk check covers point 4 by checking the bare path rather than addressing it as `<path>#`.
  Point 1 stays out of scope by the operator decision, and point 3 belongs to `loop-exit-halt-names-its-cause`.
- Issue #823: a test pins that `plan:sub_test#TestFoo` resolves against an external test package.
  A `not_found` Create or Rename new side now names the unit's files and every declaration of the same name elsewhere.
  The plan-spec line in `contracts/specs/loom-plan-spec.md` and the `doc.go` updates record the new behaviour, as the docs rule requires.

The diff makes no change the entry does not call for.

## Carried-over risks

The Darn-Review carry-over names `glyph-rejected-uncheckable-branch-untested`.
Commit `b97b9c766` adds `GlyphRejectedWithUncheckableNamesTheFreshRestart` to `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate`, which tests that branch.
No carried-over risk names a parent correction that was never recorded.
The open risk about long searched-files lists is a design detail and does not affect scope.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and two peers; `own_row` is 21.

- `loop-exit-halt-names-its-cause` (row 16, Merge-In) changes `internal/loomcli`, `internal/loomengine` and `internal/shedverbs`, none of which this diff touches.
  Its subject is the `loop-exited` halt reason, which this run's entry hands to it, so the two runs split the work and do not conflict.
  There is no overlap, so no landing order applies.
- `orch-strand-cycle` (row 5, Discussion-Burler-Review) lists no changed paths and covers orch strand lifecycle, unrelated to Webster's done-checks.
  There is no overlap, so no landing order applies.

Both `plan-card-rules` and `gate-runs-once` depend on this task and are not running, so they are not overlaps.
This run makes no board entry obsolete.
