```yaml
lens: fix-code-correctness
outcome: reported
```

Findings, traced step by step through each changed branch. I ran the tests only as far as the fixer's gate output shows.

**LOW — `TestStartTestMap`, row "a registration that fails leaves the child running and returns no error"**
- The row never asserts that `Register` failed. It assumes a run-proc dir below a regular file fails.
- If `Register` succeeded there, the row would still pass. It then exercises the same path as the first row and pins nothing about the failure branch.
- The test does fail with the old `EndProcess` call in place: the killed child never writes `done.txt`, so `awaitFile` fails.
- A cheap fix is to assert `runproc.List(runProcDir, ...)` errors or returns no record for this row.

**LOW — `TestRun_AFailingTestMapStartLeavesTheRestOfTheTick`, failure mode with the defect in place**
- With the old `return err` in `tick`, tick 1 reaches the Tick hook (count 0, no start).
- Ticks 2 and 3 return before the hook, so `h.ticks` stays at 1 and `h.cancel` is never called.
- The test then hangs until a harness or package timeout instead of failing with `ticks = 1; want 3`.
- It catches the defect, but as a hang. This is the weakest point of the F2 test.

**No other findings.** Traced inputs:
- **`testMapStep` with `test_map_every` 0:**
  - Count 3 gets zeroed through `UpdateState`; count 0 returns with no write.
  - The row asserts count 0 and no start. Without the reset the count would be 3, so the row fails.
  - Enabling the key to 2 afterwards finds count 0 and starts none.
- **`tick`:**
  - A `testMapStep` error goes to `testMapFailures.Fail` and the tick continues to `landBeam`, `announceLandings` and `Deps.Tick`.
  - Success calls `Recover`.
  - The start-attempt count is 2 (ticks 2 and 3), matching the test.
  - A move record skips the step, so no Fail or Recover runs; that is acceptable.
- **`loop_move_test`:**
  - The landing is counted on tick 2 while the record stands (`countLandings` runs regardless).
  - The refresh starts on tick 3, after the record is removed in `perTick`.
  - Dropping the `moving == ""` guard would start it at tick 2, which fails the "none while standing" assertion.
- **`beamDown`:**
  - With `dobbyLive` nil, `live` is false and `endTestMap` still runs if set.
  - A dead dobby plus a live refresh ends only the refresh, matching the test row.
  - An `endDobby` failure returns before `endTestMap`, which leaves the refresh alive. That is the same as before the fix, so it is not a regression.
  - A failing `endTestMap` returns the wrapped error with `DobbyEnded` true, matching the test row.
  - `endDobbyRoles` had no remaining callers or tests, and the removed `runproc` import in `stop.go` is unused.
- **`BeamDown` callers:**
  - `internal/beamcli/cli.go:578`, `internal/loomcli/pawlwiring.go:162` and `internal/battencli/cancelcmd.go:355` call `orchcli.BeamDown`, and each now also ends the prime's refresh.
  - `pawlwiring.go` and `cancelcmd.go` presumably pass a beam location, where `endTestMap` is nil. I did not check whether any passes the prime.
- **`startTestMap` on `Register` failure:**
  - It logs a Warn and returns nil, and the `hubDobbyDeps` wiring uses the new signature with `selfexec.LymBinary`.
  - The step then resets the count, so no second unregistered child starts on the next tick.

Calls made:
1. `grep -rn "BeamDown(\|\.beamDown()"` over non-test Go files, to find every caller of the changed `beamDown` behaviour. My context only held the `endDobbyRoles` call sites.
