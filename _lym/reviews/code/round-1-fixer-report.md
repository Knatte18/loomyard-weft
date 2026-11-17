---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The commit-first check is made as proposed (refuseWhileMergeActive before the write). The retry selection takes a finished keep intent of this machine and lets the commit's own no-op show whether HEAD lacks it, instead of reading the records HEAD; an intent already written is skipped while a merge is active, so a committed one never errors.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: The proposed td.Run also deletes and pushes the batten record of a cancelled run, which another machine owns after a hand-over (teardown.go:565, followup.go:91-108). RemoveHandedOverPair calls EndSession then RemovePair with the same Request, so the reed session ends first and the record is left alone.
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
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
    disposition: fixed-differently
    reason: A failed last-commit read is skipped as proposed, but logs through a per-slug EpisodeSet (unreadableOrphanTip) instead of one Warn per call, so a persistently broken pair logs once and not every probe tick.
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Burler fixer report, code round 1

This session found every finding already fixed and committed by an earlier fixer session of this round (commits `29f59d8c` to `3b9ec898`), with its lens forks run and their reports saved.
I re-read the changed sources (`leftoverpairs.go`, `pairteardown/leftover.go`, `dobbyengine/orphan.go`, the `dobbyengine/doc.go` bullets, the `tendercli` help) and found each fix in place and the tree clean.
I spawned no further fork: the one fork pass already ran.

The gate then failed on `reedengine` `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory`, a package this round did not change.
The test passes alone in 3s and times out at its 5s recorder wait when the package's parallel tmux tests load the machine; it failed in four of seven verify runs.
I raised the recorder waits of `internal/reedengine/bindings_integration_test.go` to a shared 30s `recorderDeadline` (commit `d85b0cb4`).
`lym gate test` over `internal/reedengine` tiers 0 and 2 then passed.
A comment-wrap gate then flagged the `recorderDeadline` comment, whose second line broke a sentence at a comma; it is now two sentences.
No finding's disposition changes.

## Changes

- F1, F11, F6, F7 (`internal/battencli/leftoverpairs.go`, `cancelcmd.go`): `refuseWhileMergeActive` is extracted from `commitCancelRecord` and runs before the kept intent is written.
  A finished keep intent of this machine whose commit failed is committed again; `MarkKeptCancels` returns the slugs it committed.
  `DobbyOrphans` skips a pair whose last commit does not read, with one episode log.
  Tests: `leftoverpairs_integration_test.go` (orphan classification, pre-upgrade marking, the retry), and `leftoverpairs_test.go` (the merge-active refusal through `writeKeptCancel`).
- F2, F15 (`internal/dobbyengine/orphan.go`): the first sight is recorded only for orphans whose notice posted; a failed notice is logged through the orphan episode and posted again at the next probe tick.
  Tests: a notice-failure row and a failing-reading row in `TestOrphans`.
- F3, F12, F5, F17 (`internal/pairteardown/leftover.go`, `doc.go`): `RemoveHandedOverPair` ends the pair's session before the local removal and returns `Result`.
  `RemoveOrphanPair` re-reads the board, the batten run directory and the slug's processes first.
  Tests: `leftover_integration_test.go` and `leftoverhandedover_integration_test.go`.
- F4 (`internal/beamcli`): `dobbyleftover_integration_test.go` pins the landed and cancelled dispatch and the workspace regeneration.
- F8, F14 (`internal/battencli`): two kept-line rows in `TestDobbyLines`; `dobbyruns_integration_test.go` reads `PairKept`, `KeptSince`, `HandOverMarked`, `OwnedElsewhere` and `BoltHeld` from records.
- F9, F13 (`internal/fabricengine`): `pairpushdeferred_integration_test.go` covers `PairPushDeferred` and `PairLastCommit`.
- F10, F16 (`internal/dobbyengine`): a done-run clear test for the `PairPresent` skip; the kept-pairs bullet moved below the runs continuation.
- F18 (`internal/tendercli/cli.go`): the lift paragraph is rewrapped near 80 columns.
- Docs kept in step with F1 and F2: `internal/dobbyengine/doc.go`, `loop.go` (`Deps.MarkKeptCancels`) and `internal/beamcli/doc.go`.

## Self-check

One fork pass ran in the earlier session; the lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, MEDIUM, `RemoveHandedOverPair` via `td.Run` deletes a handed-over cancelled run's record: fixed (EndSession then RemovePair), duplicate of the fix-proposal finding.
- fix-code-correctness, MEDIUM, `MarkKeptCancels` errors on a committed intent during a merge: fixed (an existing intent is skipped while a merge is active).
- fix-proposal, MEDIUM, the F3 `td.Run` tail: fixed, merged with the correctness finding above.
- fix-proposal, LOW, F2 re-posts the notice every probe tick while the first-sight write keeps failing: disputed, every fix worse (see Disputed).
- fix-proposal, LOW, F11 Warn per tick: fixed with an EpisodeSet.
- fix-proposal, LOW, F1 selection by committed-by-this-machine: fixed-differently, as recorded under F1; an owed push of an already-committed record is retried by the push-owed mechanism, not here.
- fix-consistency, LOW, dobbyengine `doc.go` first-sight bullet: fixed.
- fix-consistency, LOW, the three kept-cancel descriptions (`loop.go`, `doc.go`, `beamcli/doc.go`): fixed; the `markKeptCancels` log line in `internal/dobbyengine/leftover.go` keeps its wording, which a test pins and which still reads true.
- fix-consistency, LOW, `HandOver` calls `RemovePair` with no session end: deferred (see below).
- fix-affected-set, LOW, dobbyengine `doc.go` first-sight bullet: fixed, duplicate of the consistency finding.
- fix-affected-set, LOW, dobbyengine `doc.go` kept-cancel bullet: fixed, duplicate of the consistency finding.

## Disputed

- fix-proposal LOW on F2, ground `every fix worse`: a notice that posts and then a failed first-sight write repeats the notice at each probe tick until the write works.
  Recording before the notice is the order F2 removed, since it leaves an orphan torn down after a grace that ran from an unnoticed sight.
  Evidence: `internal/dobbyengine/orphan.go:81` (`postOrphanNotice` runs before `updateOrphanSights` at line 88).

## Deferred

- fix-consistency LOW, `HandOver` (`internal/pairteardown/handover.go:147`) removes the pair with `RemovePair` and no session end.
  No finding of the review names that path, it covers a caller outside the pair with its own tests, and the fix-check rubric excludes a further improvement the fix did not make worse.
  Whether `HandOver` should end the session first is a separate change for the operator or the next round.

## Rulings

None reached this round.
