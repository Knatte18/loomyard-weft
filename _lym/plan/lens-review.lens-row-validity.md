```yaml
lens: row-validity
outcome: reported
```

Row-validity review: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

**Findings**

1. **MEDIUM** — Card 12, `RunGate` subtest of `TestGateTest_Scopes`: the "`Started` once per tier" assertion can fail because a tier may be carried green.
   - Evidence: the card says a carried tier "ends with no start" through `reportCarried` → `report`.
   - The subtests share one pair in sequence, and their siblings `edit(...)` first to break the carry (`internal/gatecli/scopes_integration_test.go:508-515`).
   - The planned subtest names no edit, so a green record left by an earlier subtest carries the tier and `Started` never fires.
   - Fix: edit a file first, as the sibling subtests do.

2. **MEDIUM** — Card 12: no row reaches `Ended` for a carried tier or for `reportBlocked`, though card 13's record depends on `Ended` for every tier.
   - Evidence: card 12 Intent, "a carried tier … ends with no start; `reportBlocked` calls `Ended`".

3. **MEDIUM** — Cards 14 and 15: `checkAlive` is never tested, and the "start ticks differ" rows repeat the dead-pid rows.
   - Evidence: `ensureCheck` and `checkView` take `alive` as a seam or bool (card 14 `plan:internal/adoptcli#ensureCheck`, card 15 `plan:internal/adoptcli#checkView`).
   - So the "differing ticks" row in `TestEnsureCheck` and in `TestCheckView` drives the same `alive=false` input as the dead row.
   - `checkAlive`'s own branches have no row: pid alive with equal ticks, with differing ticks, with no ticks, and a dead pid.
   - Decision 11's reboot-reused-pid case therefore goes untested.
   - Fix: add a tier0 `TestCheckAlive` over `os.Getpid()` with its real ticks, wrong ticks and no ticks, plus a dead pid. Drop the repeat rows.

4. **MEDIUM** — Card 7, row "re-exec record whose record a goroutine rewrites to another pid during the wait posts nothing": the row can pass without the second-read branch existing.
   - Evidence: the fresh-refused path and the second-read path give the same output, the refused line and no post (card 7 steps 3 and 5).
   - If the rewrite lands before `Run`'s first `ReadsAsReexec`, the start reads as fresh, and code with no second read also passes.
   - Fix: give the rewrite a fixed point well inside a long `ReexecLockWait`, for example at half of a one-second wait, so the first read has already passed the re-exec test.

5. **MEDIUM** — Card 8: the failed-stop row needs an error field on `tidyShuttleFake`, and the card lists that fake only under `Uses`.
   - Evidence: `internal/dobbyengine/tidy_test.go:32`, where `StopStrand` always returns nil.
   - Fix: list `internal/dobbyengine#tidyShuttleFake` and its `StopStrand` under `**Edit:**` with a `stopErr` field.

6. **MEDIUM** — Card 8: no row reaches the rule that replaces the kept `State.Tidy` after a tick that ends with the marker present.
   - Evidence: the Tidy-in-flight row seeds the Tidy before start, so `recordStart` alone supplies `keptTidy`.
   - Code that never replaces the kept value after a tick passes every row.
   - Fix: add a row where the record gains a Tidy on tick 1 with `.git` present, and the worktree is removed on tick 2. Assert `StopStrand` of that guid.

7. **MEDIUM** — Card 6 / card 10: nothing tests that `SpawnDetached` sends the child's output to `OutPath`.
   - Evidence: `SpawnDetached` returns `ErrSpawnUnderTest` in a test binary (`internal/dobbyengine/start.go:261`).
   - Card 10's tier1 test starts `<copy> beam dobby` with its own redirection, not through `SpawnDetached`.
   - The criterion "a refusal with no resolved worktree, in a process this binary spawned, leaves its envelope in `dobby.out` and nothing in `dobby.log`" is therefore unreached.

8. **MEDIUM** — Card 10, `TestDobby_ReexecsOntoAReplacedBinary`: two gaps.
   - It states no skip off the platforms where `selfexec` execs, so on Windows, where `statIdentity` always fails (`internal/selfexec/identity_windows.go`), the test cannot pass.
   - The `Tidy` it writes states no future `Deadline`, so the dobby's `settleTidy` times it out at once (`internal/dobbyengine/tidy.go`, the `now.Before(tidy.Deadline)` check) and clears it. The assertion that the record still holds the `Tidy` then fails.
   - Fix: skip on Windows, and seed `Deadline` far in the future.

9. **LOW** — Card 7, "a failing `CheckBinary` … one notice" row: the fake must call `failed` once only.
   - The once-per-identity bound lives in `selfexec`, which the loop fake replaces; a fake that fails on every call posts once per iteration up to `stopAt`.

10. **LOW** — Card 7, the "released after `lock.ProbeGrace` plus 150 ms" rows depend on wall-clock timing.
    - A fresh start must give up before the release, which holds only while `Run` starts acquiring within 150 ms of the timer. That margin is thin under a loaded gate, against the scribe:testing rule against wall-clock dependence.
    - Fix: start the release timer from inside a seam `Run` calls just before acquiring, or widen the margin.

11. **LOW** — Card 8: the exit and stop lines go through `Deps.Log`, which card 7 makes a recording fake in `newLoopHarness`, yet the rows assert "the prime log holds" them. The card should say which of the two the row reads.

12. **LOW** — Card 11, the `--gate adopt` subtest of `TestGateTest_Scopes` repeats the `finalize` subtest (`internal/gatecli/scopes_integration_test.go:508`).
    - The scope-`all` path is key-generic, and the key's presence is already pinned by `templateDefaults` in `TestLoadConfig`. It is kept only as the record's acceptance row.

13. **LOW** — Card 4, the "Tidy in flight" row states no future `Deadline`.
    - `step()` runs `tidyStep` after the count, so a zero deadline takes the timeout path in that same step, and the row's "until the tidy settles" stage is not set up as written.

14. **LOW** — Card 3: the new branch in the `hubDobbyDeps` `Landings` wiring, where a failed `machinename.Name` returns its error, has no row.

15. **LOW** — Card 10, the binary-replacing test: it states no cleanup that kills the detached dobby when the test fails before SIGTERM, against scribe:testing's "Leave nothing behind".

**Tool calls**
- Read `internal/dobbyengine/passes_test.go:80-235`: the pass harness and `TestLog_OneLinePerStartAndPass` body, which the context lacked.
- Read `internal/dobbyengine/tidy_test.go:1-130`: the tidy harness and `tidyShuttleFake`, not read before.
- Grepped `internal/gatecli/scopes_integration_test.go` subtests and `internal/hubgeom/dobbygeom_test.go`: the gate subtest names and the `TestDobbyGeometry` shape were unread.
- Read `newPreDriveReceiver`, `seedBattenRun` and the `TestMigrateConfig` head: the fixtures cards 2, 3 and 11 build on were unread.
- Read the `--gate finalize` subtest and `twoTier`, and grepped `loop_test.go` for `Watch`: needed for the carry and `TiersValue` checks.
- Read `hubgeom.ParentOf`: to confirm card 3's pair-less runs count as the prime's.
