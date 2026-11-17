# Fixer-report — code review round 1

All nine findings are fixed. `lym gate test` passes at tier 0 for `internal/loomcli`, `internal/shedrun` and `internal/stencilstore`, and at tier 1 for `internal/loomcli`.

## Changes

- **F1 (fixed as proposed, in a different shape).**
  `bootstrapCommitPaths` now adds `shedrun.StencilPinsRel` whenever the pins file is on disk, not only when this call wrote it.
  The first-seed write stays gated on `seedErr == nil`.
  The review put the stat in `seedAndCommitBootstrap`; the self-check showed that left the caller's wiring untested, so the stat sits inside `bootstrapCommitPaths`, where its tier-0 test pins it.
  `writeStencilPins` no longer returns a bool, since nothing reads it.
- **F2 (fixed as proposed).**
  `stencilsChangedNotice`, the start and resume long help, `loomcli/doc.go` and the `stencilpins.go` header now say `lym stencil diff` inspects each stencil against its shipped default and its board copy.
  None claims it shows the change since the run started.
- **F3 (fixed differently).**
  Proposed: a start test that runs the verb to success with a recording `notifyParent`.
  Not made: the start `RunE` needs a live reed (`c.reed.Up`), so no tier-0 test reaches its success path; the only test that does is a tier-2 smoke test over a real tmux.
  Instead: the success envelope is built by the new `startSuccessFields`, and `TestStartSuccessFields` asserts `stencils_changed` and exactly one notice for an edited stencil, and none for an unchanged stencil or a driver left alone.
- **F4 (fixed differently).**
  Proposed: a git-backed bootstrap test asserting the pins file is tracked after the first seed, or the lighter path-equality check.
  Made: both halves of the lighter option.
  `TestStencilPins_RoundTrip` asserts that `AnchorPath` joined with `StencilPinsRel` equals `StencilPinsFile`.
  `TestBootstrapCommitPaths` has rows that pin the pins record's presence and absence in the pathspec, over a real file in a temp directory.
  Not made: a test that drives a real commit, since it needs a git-backed pair the file does not have.
  A dropped `CommitAnchoredPaths` argument is therefore not caught.
- **F5 (fixed as proposed).**
  A row in `TestResumeVerb_NamesStencilsChangedSinceTheRunStarted` wakes a parked live driver and asserts `stencils_changed` and one notice.
- **F6 (fixed as proposed).**
  A row in the same test returns an error from `notifyParent` and asserts the envelope is ok and still lists the edited stencil.
- **F7 (fixed as proposed, with one deviation).**
  `TestChangedPins` has a row that edits three stencils and wants them in name order, and a row for an unreadable board copy that asserts an error naming the stencil.
  The unreadable copy is a directory where the file belongs, not a `permkit.Chmod`, so the row works when the tests run as root.
  The fixture pins a third stencil, so an unsorted result fails on five of six map orders.
- **F8 (fixed as proposed).**
  `runDriverSpawnAndWait` returns a second result, true for a spawn or a woken parked driver.
  `startSuccessFields` warns only when it is true.
  The start long help and `loomcli/doc.go` say so.
  `TestRunDriverSpawnAndWait_*` assert the second result for the spawn, the parked wake and the live-driver no-op.
- **F9 (fixed as proposed).**
  `resumeFromRunState` has a `succeedAfterWake` closure that releases the bootstrap lock before `warnChangedStencils`; both wake paths use it.

Commits: `a7987cdb` (F7), `c576bcd7` (F1, F4), `159fc358` (F2, F3, F5, F6, F8, F9) and a final commit for the self-check follow-ups.
F2, F3, F5, F6, F8 and F9 share one commit because they edit the same files (`start.go`, `stencilpins.go`, `stencilpins_test.go`) in overlapping hunks.
The other commits follow the per-finding format.

A gate then flagged six wrapped or over-long comment lines the round added, in `sharedbootstrap.go`, `start.go`, `stencilpins.go` and `stencilpins_test.go`.
Each is now one sentence per line, and a further commit carries the repair.

## Self-check

- fix-code-correctness, LOW: the caller's `pinsStatErr == nil` wiring of the pins commit is untested. Fixed with my own change: the stat moved into `bootstrapCommitPaths`, whose tier-0 test pins it.
- fix-code-correctness, LOW: the ordering row with two names passes unsorted half the time. Fixed as the lens proposed: three names.
- fix-affected-set, LOW: the `stencilpins.go` header still said only a resume warns. Fixed as proposed.
- fix-consistency: no findings.
- fix-proposal: no findings; its note on F4 is the first fix-code-correctness finding above, which is now fixed.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
The real-git commit test of F4's first option was not made; see F4 above.

## Rulings

None reached me.
