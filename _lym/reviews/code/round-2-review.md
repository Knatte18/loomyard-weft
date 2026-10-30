---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/dobbyengine/runnotice_test.go:351
    summary: The restart rows read the notice memory back only for the revive key; the episode since and the revive streak count are saved but a test never needs them read back
    fix: In TestRunNotice_OncePerEpisode, set f.child.Modified = h.clock.now() (a status write with the child still running) just before h.restartDobby(), and keep the assertion that the restarted dobby sends no second notice; it fails when the memory's Since is not read back, since the driver-dead key stamp then moves and the key is dropped. In TestRunNotice_APIErrorRevive, add a row that calls h.restartDobby() after the second revived episode and before the third, and expects one notice on the third; it fails when APIRevives is not read back.
  - id: F2
    severity: LOW
    class: design
    location: internal/beamcli/dobbydeps.go:229
    summary: "runShedDuration's new fallback for a zero or negative count has no test; the integration test only reads the recipe's explicit 30"
    fix: Add a tier0 table test in internal/beamcli for the count-to-duration step, by moving the int, int64 and float64 switch into a small function over the row's Config value that runShedDuration calls, with rows for a positive int, int64 and float64, zero, a negative count and an absent key expecting the fallback. Alternatively state in the test file that the fallback is pinned by the integration test, and add a row there that builds the deps from a recipe whose Run-Shed row names notice_quiet_min 0.
---

### [LOW] The restart rows pin the notice memory's read-back only for the revive key

**Location:** `internal/dobbyengine/runnotice_test.go:351`

**Issue:** The focus question is whether a test fails when the notice memory is not saved or not read back across a re-exec, for the episode marker and the revive streak.
Save is covered: `AReviveAndTheEpisodeMemorySurviveARestartThroughTheStateRecord` asserts the recorded `RunNoticeState`, and a dobby that never saved it would revive twice, so it fails.
Read-back of `APIReviveKey` is covered by the same row (the restarted dobby revives nothing more).
Two fields are not: the restart in `TestRunNotice_OncePerEpisode` keeps `child.Modified` unchanged, so a dobby that dropped `Since` on read-back would rebuild the same `Since` from `child.Modified` and still send nothing, and no test restarts inside a revive streak, so `APIRevives` read-back is never asserted.
Both gaps matter in production: a child's status file is rewritten while it stays running, which moves `Modified`, so a lost `Since` re-sends driver-dead after a re-exec, and a lost streak count delays the third-episode notice.
The marker half of the question is covered: `OncePerEpisode` asserts the marker's keys and a restart over the same marker.

### [LOW] runShedDuration's zero and negative fallback is untested

**Location:** `internal/beamcli/dobbydeps.go:229`

**Issue:** The focus question is what an explicit zero `notice_quiet_min` does now compared with before the change.
Before: `innerRunEntry` turned zero into `defaultInnerRunNoticeQuietMin` (30), so zero meant 30 minutes.
Now: `innerRunEntry` validates the key and passes it on, and `runShedDuration` turns a non-positive count into `defaultNoticeQuiet` (30 minutes), so zero still means 30 minutes; the behaviour is unchanged and the `entries_batten.go` and battenshed docs state it.
`runShedDuration` runs only from `hubDobbyDeps`, and the one test that reaches it reads the shipped recipe, whose row names 30; no test passes a zero, a negative count or an absent key, so the stated fallback is not pinned.
The `Quiet == 0` rows in `runnotice_test.go` pin dobbyengine's own reading of zero, which the wiring never produces.
The same helper now also gives `notice_probe_s` the default for a non-positive count where the old reader returned a zero interval; that follows the same rule and shares the gap.

## Verdicts on the focus questions

- Run selection: the cancel marker, a non-running batten state, an absent child status, a `ReadChild` error and a nil `ReadChild` each have a row in `TestRunNotice_Conditions` with a dead driver, so removing any one skip sends a notice and fails the row. A `ReadChild` error leaves the tick succeeding, because `h.tick` fails the test on a tick error. Not a defect.
- `RespawnCapReached` and `readNoticeChild`: a status write after the last respawn reads as not capped in `RespawnCapReached`, and `TestReadNoticeChild` has a row for it and one for an unreadable memory record, which reads as not capped while the child is still read, so no condition of the run is hidden. Not a defect.
