## Changes

Every finding is fixed; the commits are on `notices-from-dobby`, none pushed.

- F1, F2, F7, F9 (one commit, `internal/dobbyengine/runnotice_test.go`): fixed as proposed.
  - F1: a restart row in `TestRunNotice_APIErrorRevive` reads the persisted `RunState.Notice` fields back, then replaces the daemon and asserts no second revive and a notice at the next sight.
  - F2: rows for a non-running run state, a cancel marker, an absent child status, a `ReadChild` error and a nil `ReadChild`.
  - F7: two zero-quiet rows, one for the no-run fallback and one for the agents-read path.
  - F9: the delivery retry test asserts the dropped notice's record is kept while its run is listed and gone once the run leaves the listing.
  - A `restartDobby` helper now serves this and `TestRunNotice_OncePerEpisode`.
  - Mutation runs confirmed the new rows fail with `saveNoticeMemory`, the delivery cleanup, the cancel-marker skip, the batten-state skip or the `quiet > 0` guard removed.
- F3: fixed with an own change to the signature.
  - `RespawnCapReached` takes the child status file's modification time as a parameter, where the review proposed reading the file inside.
  - `readNoticeChild` already stats that file, so passing its time avoids a second stat and a race.
  - The behaviour is the proposed one: a status write after the last respawn reads as not capped.
  - `readNoticeChild` treats a memory read error as not capped and logs it at Debug.
  - The existing respawn-cap test gained a false-after-write assertion.
- F4: `battenshed.RespawnCap` exports the one constant, and `ChildReading.RespawnCap int` replaced `RespawnCapped bool`.
  - Zero means not capped, and the driver-dead text renders the count from the reading.
  - The `dobbyengine` constant is gone.
- F5: fixed as proposed. `runShedDuration` returns the fallback for a count that is not positive.
- F6: fixed as proposed. `resetText` and `resetStamp` are deleted from `internal/battenshed/notice.go`.
- F8: fixed with an own change.
  - The new `internal/battencli/dobbynotice_test.go` maps the status file and memory record into `ChildReading` over a fake task worktree.
  - The lock fallback is extracted into `childDriverLockHeld`, so its rows (held driver lock, held run lock, unreadable lock, none held) run without a reed session.
  - The review proposed rows on `noticeDriverAlive` itself, which would need a live reed status read for the strand step before the locks.

## Self-check

- fix-consistency, LOW: `innerRunEntry`'s doc did not say a zero `notice_quiet_min` means the default. Fixed in `internal/shedrecipe/entries_batten.go`.
- fix-affected-set, LOW: `internal/battenshed/doc.go` still described `RespawnCapReached` as reading only the count. Fixed.
- fix-proposal: no findings.
- fix-code-correctness: no findings.

Both fork findings went in the commit "battenshed: fix self-check". No fork returned without a report.
The lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
