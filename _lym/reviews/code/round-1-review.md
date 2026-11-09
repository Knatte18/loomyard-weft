---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/beamcli/dobbydeps.go:585
    summary: startTestMap ends an unregistered child through runproc.EndProcess, which PATTERN-child-processes reserves for the orphans lym tender reap ends
    origin: lens:code-affected-set
    fix: In startTestMap, delete the runproc.EndProcess call in the Register failure branch. On a Register failure log one Warn naming the pid and the error, leave the child running, and return nil. The dobby already waits on the child in the background, and it ends on its own within the verb's timeouts. Returning nil resets the count, so the next tick does not spawn a second unregistered child. Update the doc comment on startTestMap and the matching line in internal/beamcli/doc.go ("A registration that fails ends it again") to say an unregistered refresh is left to end on its own. Add the Register-failure case to the startTestMap test of F4.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/loop.go:648
    summary: any testMapStep error returns from tick, so landBeam, announceLandings and the Deps.Tick hook (hibernate carrier, Slack relay) are skipped every tick while an optional refresh keeps failing
    origin: lens:code-correctness
    fix: In tick, stop returning the error of d.testMapStep(now). Report it once per episode through the same Fail and Recover pair that d.moveStands uses (add a sibling field on daemon for the test-map step, logging via logger with the worktree and the error), then carry on with landBeam and the rest of the tick. Keep testMapStep returning its wrapped errors so the retry and the kept count still hold. Update the testmap_test.go failed-start subtest to assert on the step's returned error as now, and add one tick-level row showing a failing StartTestMap leaves landBeam reached.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/orchcli/stop.go:57
    summary: beamDown returns before endDobby when the dobby is not live, so lym beam stop of the prime leaves a running test-map refresh alive when the dobby has died
    origin: lens:code-correctness
    fix: In internal/orchcli, split the refresh end out of endDobbyRoles into its own step that beamDown runs for the prime whether or not the dobby is live, after the dobby end when one ran. For example give orchCLI an endTestMap func, set in wireCLI only when owner is runproc.HubOwner and nil for a beam, and call it in beamDown on both the live and the not-live path. A failure of it is returned wrapped, as the dobby end is. Update TestEndDobbyRoles (or its replacement) with rows for a dead prime dobby with a live refresh, and for a failing refresh end. Adjust internal/orchcli/doc.go and internal/beamcli/doc.go to say the refresh is ended on a prime stop regardless of the dobby.
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/beamcli/dobbydeps.go:565
    summary: startTestMap has no test, so its argv, its working directory, its RoleTestMap registration under HubOwner and its handling of a failed registration are unpinned
    origin: lens:code-tests
    fix: Make the binary lookup of startTestMap a parameter (selfexec.ResolveLymBinary over an injectable lookup, as the package's other spawners do) and add a test in internal/beamcli that passes a stand-in executable (a tiny script or the test helper process pattern the package already uses) and a temp run-proc dir. Assert the child ran as gate testmap --max N in the prime anchor, that runproc.List(runProcDir, runproc.HubOwner) shows one record with Role runproc.RoleTestMap, and, with an unwritable run-proc dir, the behaviour F1 settles. The test runs at the lowest tier that fits, launches no test binary and no go test, and uses t.Parallel.
  - id: F5
    severity: LOW
    class: design
    location: internal/dobbyengine/loop.go:644
    summary: no test shows that the tick skips the test-map step while an anchor-move record stands
    origin: lens:code-tests
    fix: In the dobbyengine test that already sets Geometry.MoveRecords for the tidy step (loop_move_test.go), add a StartTestMap fake and a count at the threshold, and assert it is not called while the record exists and is called once the record is gone.
  - id: F6
    severity: LOW
    class: consistency
    location: internal/dobbyengine/doc.go:447
    summary: a prime anchor move waits on the refresh as a pass in flight and then ends it through EndSparing, and the refresh is not retried until another test_map_every landings; the dobbyengine doc bound and the fabriccli doc say neither
    origin: lens:code-affected-set
    fix: Add one bullet to the test-map refresh Bound list in internal/dobbyengine/doc.go saying that an anchor move of the prime counts a running refresh as a pass in flight, waits for it, and ends it with the dobby's other processes, so what it was building waits for the next test_map_every landings. State the same in the fabriccli doc beside the pass-in-flight and EndDobbies text. If the refresh should instead survive an anchor move, add runproc.RoleTestMap to the spare list at internal/fabriccli/anchor.go:468 and say so in both docs.
  - id: F7
    severity: LOW
    class: design
    location: internal/dobbyengine/tidy.go:143
    summary: the refresh count grows without bound while test_map_every is 0, and enabling the key later starts a refresh at once on the stale count
    origin: lens:code-correctness
    fix: In countLandings add to LandingsSinceTestMap only when the wired seams exist and the loaded test_map_every is above 0, or reset the count to 0 in testMapStep when test_map_every is 0. Add a row to the test_map_every 0 subtest of TestTestMapStep asserting the count stays 0 after landings, and update the countLandings doc comment sentence on the refresh count.
  - id: F8
    severity: LOW
    class: design
    location: internal/beamcli/dobbydeps.go:217
    summary: nothing pins that only the prime's hubDobbyDeps fills TestMapRunning and StartTestMap, nor that countLandings adds nothing to the refresh count while a tidy agent is in flight
    origin: lens:code-tests
    fix: In the existing hubDobbyDeps test that builds both the prime's and a beam's deps (dobbydeps_integration_test.go), assert TestMapRunning and StartTestMap are non-nil for the prime and nil for the beam. In TestTestMapStep add a row with State.Tidy set and fresh landings, asserting LandingsSinceTestMap stays unchanged.
  - id: F9
    severity: LOW
    class: design
    location: internal/orchcli/beamdown_test.go:25
    summary: the error returns of testMapStep (config load, liveness read) and the failure of the refresh end in the stop are untested
    origin: lens:code-tests
    fix: Add a failRole of runproc.RoleTestMap row with wantErr to the stop test, shaped as F3 settles it, and add rows to TestTestMapStep where LoadConfig fails and where TestMapRunning fails, each asserting the wrapped error text and that the count is kept and nothing is started.
---

### [BLOCKING] startTestMap ends its child through runproc.EndProcess

**Location:** internal/beamcli/dobbydeps.go:585
**Issue:** `PATTERN-child-processes` says `EndProcess` serves only the orphans `lym tender reap` ends, and that a site signalling a run's process by another route is a review finding. The only other caller of `runproc.EndProcess` in production code is `internal/tendercli/deps.go:117`. No commit carries a `Pattern-Departure:` trailer. The constraint is violated whatever the reason, so it blocks.

### [MEDIUM] A testMapStep error aborts the whole tick

**Location:** internal/dobbyengine/loop.go:648
**Issue:** On the prime, a failing refresh (unresolvable binary, unreadable run-proc dir, failing registration) returns from `tick` before `landBeam`, `announceLandings` and the `Deps.Tick` hook, which carries the hibernate carrier and the Slack relay (`dobbydeps.go:213`). The step is optional, and its failure should not stall these. `tidyStep` has the same shape, but that does not make this correct.

### [MEDIUM] Stopping the prime leaves the refresh alive when the dobby is gone

**Location:** internal/orchcli/stop.go:57
**Issue:** `beamDown` returns at `if !live` before `c.endDobby()`, and the refresh end sits inside `endDobbyRoles`, so it never runs for a dead prime dobby. The Goal asks that the child is ended on stop. The docs cover only a dobby that dies without a stop.

### [MEDIUM] startTestMap is untested

**Location:** internal/beamcli/dobbydeps.go:565
**Issue:** The record's Stop and single-flight criteria rest on its registration under `HubOwner` as `RoleTestMap`, and the `--max` argument is the config's whole effect. `testmap_test.go` fakes `StartTestMap`, and `TestTestMapRunning` pins only the read side.

### [LOW] The tick's skip of the test-map step during an anchor move is unpinned

**Location:** internal/dobbyengine/loop.go:644
**Issue:** `doc.go` and the `tick` comment both claim the skip. No test sets a move record with the refresh wired.

### [LOW] The anchor move's effect on the refresh is undocumented

**Location:** internal/dobbyengine/doc.go:447
**Issue:** `dobbyPassInFlight` (`internal/fabriccli/anchordeps.go:489`) reads the refresh as a pass in flight because it is a direct child of the dobby, so the move waits for a long build-and-test run. `EndDobbies` then uses `EndSparing(RolePush, RoleSync)` (`anchor.go:468`), which ends the `test-map` record too. The count was reset when the refresh started, so what it was building waits for another `test_map_every` landings. The change description states the wait but not the end, and neither doc carries either.

### [LOW] The refresh count grows while test_map_every is 0

**Location:** internal/dobbyengine/tidy.go:143
**Issue:** The count adds `len(fresh)` whenever `StartTestMap` is wired, with no cap. Setting the key from 0 to 5 later starts a refresh on the next tick on landings counted while it was off.

### [LOW] Prime-only wiring and the tidy-in-flight pause of the count are unpinned

**Location:** internal/beamcli/dobbydeps.go:217
**Issue:** The record asks that only the prime's dobby takes the step, and the `doc.go` bound says no landing is counted while a tidy agent is in flight. No test asserts either for the new field or seams.

### [LOW] Error paths of the step and of the refresh end are unpinned

**Location:** internal/orchcli/beamdown_test.go:25
**Issue:** `failRole` covers only `RoleDobby`, so a failed refresh end is not shown to surface. `testMapStep`'s `LoadConfig` and `TestMapRunning` error returns have no row.

## Rejected

- Scope: `internal/orchcli/{cli.go,stop.go,doc.go,beamdown_test.go}` outside the record's three packages. The Goal asks that the child is ended on stop through `runproc.EndRole`, and the prime's stop is `lym beam stop` through `orchcli.BeamDown`. The change description discloses the placement, and a dobby's own shutdown gets only `runproc`'s grace before SIGKILL. Licensed, not a defect; F3 stays in the same package.
- Lens `code-correctness`, a `testMapStep` start that succeeds with the final `UpdateState` failing, so a second refresh starts when the first ends: the window is one failed state write, and the failure is returned and logged by the tick.
- Trigger, single-flight, prime-only gating, binary and config keys: checked against the Goal and found as asked. The step counts landings through `countLandings` as the tidy does, 0 never refreshes, no prime-HEAD record, exit record or `test_map` on/off key survives in code, config, template or docs, `--max <test_map_batch>` is passed with default 5, `testMapRunning` reads `runproc.List` under `HubOwner` and so prunes a stale record and survives a dobby restart, the spawn uses `selfexec.LymBinary`, and the seams are filled only when `geom.Slug` is empty.
- Test economy: no new test duplicates an existing one, and none launches a test binary or `go test`.
- Summary and commits: the change description's claims bear out against the diff. No commit carries a `Pattern-Departure:` trailer, and the constraints section declares no PATTERN change.
