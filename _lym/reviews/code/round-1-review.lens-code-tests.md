```yaml
lens: code-tests
outcome: reported
```

I reviewed the diff's tests under the code-tests lens.

**What each changed behaviour is pinned by**
1. `firstPassExited` reads a `PassExitOriginWait` exit as an owed catch-up (`starts.go:362`), with no notice and no start record. Pinned by the `TestStartReady_RefusalsAndBackoff` subtest "an origin-wait first pass is no refused start…". Its checks `len(notices) != 0` and `Starts["a"]` found would fail if the exit fell through to `refusedStart`.
2. The early return in `startReady` while the catch-up is owed (`starts.go:94`). Pinned by the same subtest. On tick 2 `drivePasses` reads the exit, which is buffered (`passes_test.go:168-177`), and without the guard `startReady` would spawn again that tick, so `len(spawned) != 1` would fail.
3. The task starts once the catch-up succeeds. Pinned by the same subtest (`len(spawned) != 2` after `originDown = false`).
4. The catch-up wait records the reason `origin` from the first pass. Pinned by the same subtest (`stateRecord().CatchUp.Reason`).
5. `Since` carries over consecutive failures of the same reason (`catchupwait.go:84-88`). Pinned by `TestPushSummary/ACatchUpWaitingOnOriginIsSummarisedPastTheThreshold`. Without the carry, `since` would be the 10m `At` and the notice would read "for 0m", failing `wantNotice(0, "...for 10m")`.
6. The push summary counts an origin catch-up wait as a network wait and starts the period from it (`pushsummary.go:95-99`). Pinned by the same subtest (`tickAt(9,0)` and `tickAt(10,1)`).
7. The all-clear comes once the catch-up succeeds. Pinned by the same subtest (`notices[1] != allClear`).
8. `Since` resets when the reason changes (the `s.CatchUp.Reason == reason` check at `catchupwait.go:85`). No test pins it.
9. `waitingSince` falls back to `At` for a record without `Since` (`catchupwait.go:41-46`). No test pins it.
10. The push summary leaves out a catch-up wait whose reason is `refused` or `failed` (`pushsummary.go:95`). No test pins it.
11. A `PassExitOriginWait` first pass on a dobby whose `Deps.CatchUp` has no `PullBolt`, where `oweCatchUp` does nothing. No test pins it.

**Findings**

- **LOW, design** — `internal/dobbyengine/catchupwait.go:41`
  - The `waitingSince` fallback to `At` for a record written before `Since` existed has no test, though the change description states it as a behaviour.
  - Every new write sets `Since`, and no test writes a `CatchUpWait` without `Since` and reads it through the push summary. `view_test.go:205` builds one but only compares the view.
  - So a fallback that returned the zero time would pass every test, and the summary's minutes would be wrong.
  - Fix: add a `TestPushSummary` row that seeds the state record with `CatchUpWait{Reason: origin, At: start}` and no `Since`, then expects "for 10m" at the 10m tick.
- **LOW, design** — `internal/dobbyengine/catchupwait.go:85`
  - Nothing pins `Since` resetting when the reason changes.
  - Dropping `&amp;&amp; s.CatchUp.Reason == reason` passes every test, and the wait would then inflate an origin wait with the time of an earlier `failed` or `refused` episode.
  - The rows at `loop_catchup_test.go:191` and `:221` assert only `Reason` and `Detail`.
  - Fix: add a row that fails with `failed` and then with `origin`, and asserts `Since` equals the time of the first origin failure.
- **LOW, design** — `internal/dobbyengine/pushsummary.go:95`
  - No row asserts that a catch-up waiting as `refused` or `failed` is left out of the push summary.
  - Removing the `Reason == CatchUpWaitOrigin` filter passes every test.
  - Fix: add a `TestPushSummary` row in which a refused catch-up past the threshold posts no summary.
- **LOW, design** — `internal/dobbyengine/starts.go:362`
  - When `Deps.CatchUp.PullBolt` is nil, an origin-wait first pass records nothing (`oweCatchUp` is a no-op at `catchupwait.go:51`). Nothing then stops the next tick from starting the task again.
  - No test covers this branch. The new subtest always sets `fakeCatchUpDeps`.
  - Whether the behaviour is right belongs to the correctness lens; for this lens it is a changed behaviour with no row.

Tool calls:
1. Grepped the harness helpers `newPassHarness`, `tick`, `newSummaryHarness`, `tickAt`, `wantNotice` and `fakeCatchUpDeps`: my context lacked the harnesses the new subtests run on.
2. Grepped the tests for `CatchUpWait{`, `Since` and the `refused`/`failed` reasons, and read `passFake.spawn`: I needed to know whether existing rows cover the uncovered behaviours and whether a spawned pass's exit lands in the same tick.
