# Fixer-report — code review round 1

## Changes

- F1 (`internal/pawlengine/beamawait.go`): fixed as proposed.
  `refuse` returns `awaitWake{acted: true}` when the lifting status write fails, so the wait returns to the floor.
  The `refuse` doc comment and the `awaitWake.acted` comment say so, and the `FailingBeamAwaitWriteAfterARefusal...` row pins `wantSleeps: seconds(1, 1)`.
- F2 (`internal/pawlengine/beamawait.go`, `landrequest.go`, `doc.go`): fixed as proposed, in the await marker.
  The marker carries the line `dobby-down-notified` once the notice is posted;
  `noticeDownDobby` skips the notice while the line stands, clears it when a re-entry sees the dobby live, and `refuse` clears it after a successful restart.
  Rows added to `TestBeamAwait` cover the second re-entry posting nothing, the live-again clear, both refusal outcomes and a notice that could not be posted.
- F3 (`internal/orchengine/launch.go`, `internal/orchcli/start.go`): fixed as proposed, plus the stale twin in `internal/orchcli/startdobbies_test.go` the self-check found.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW, the await-marker twins in `doc.go`, `AwaitMarkerName` and `Paths.AwaitMarker`: fixed as proposed.
- fix-affected-set: no findings.
- fix-code-correctness, LOW, a failed post is still recorded: fixed; `notify` reports success and the record is written only on success.
- fix-code-correctness, LOW, stale `acted` comment: fixed as proposed.
- fix-code-correctness, LOW, record and clear paths unpinned: fixed; rows and a `record` file check were added.
- fix-proposal, MEDIUM, F1 loops the re-check each second under a persistent write failure: fixed as proposed, no change beyond F1.
  A persistent store failure also fails the Parent-Align write in `accept`, which returns `awaitWake{}` and keeps the backoff, so the loop needs the Parent-Align write to succeed while the Beam-Await write fails.
  The review's proposal stands, and the alternative of retrying only the lift is a larger change for that narrow case.
- fix-proposal, LOW, `refuse` clears the record after a failed restart: fixed; it clears only after a successful restart.
- fix-proposal, LOW, an episode between two re-entries is unobservable: fixed with the comment on `noticeDownDobby`, which now says the record holds until a re-entry sees the dobby live.
- fix-proposal, LOW, the record is written in two steps: fixed; `recordDownNotice` writes the marker in one write.
- fix-proposal, LOW, stale comment in `internal/orchcli/startdobbies_test.go`: fixed.
