---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/testmap.go:486
    summary: A pass that ends on an error or a terminating signal leaves its unmeasured jobs out of the exit record's queued count, so work is lost until HEAD moves
    fix: In mapPackages, when the job loop returns an error (the ctx.Err() check at the loop top, a readHead error, or the error from measureJob), add len(jobs)-i to r.queued before returning, since the failed job and every later one stay unmeasured. When planPackage returns an error before the loop, add one to r.queued so the record never reads as finished work at the planned HEAD.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap.go:480
    summary: A pass on a hub with fewer than two slots (plans, counts every job queued, takes no slot) has no test row
    fix: In internal/gatecli/testmap_integration_test.go add a --upkeep row on a hub whose gate.yaml says slots 1 with a mapped tier and a queued package; assert no coverage build ran, no slot was taken, the map stands at HEAD and the exit record's Queued equals the job count.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap.go:551
    summary: After a retake, the held-lock skip and the re-plan at H before another test are not pinned, since TestGateTestMapPassYields asserts only exit 0 and the PID
    fix: In TestGateTestMapPassYields assert that a map a hand call wrote while the pass waited is not overwritten by the retake. Add a row where a hand call holds the package's build lock when the pass retakes its slot, and assert the package is skipped with the cause "another call holds its map build lock" and counted queued.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmapyield.go:79
    summary: The HEAD read on each wake of the slot wait (D8 acceptance, a landing ends a pass at a slot-wait wake) has no test
    fix: In internal/gatecli/testmap_integration_test.go add a row where the test holds the slot so the pass waits, commits to the prime, then frees the slot; assert exit 0, the package counted queued, and no map written at the new HEAD.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmapyield.go:88
    summary: A named call's wait for a build lock a pass holds, and its whole rebuild after the wait, has no test (D9 acceptance)
    fix: In internal/gatecli/testmap_integration_test.go add a row where the test holds a package's build lock, starts a named call, releases the lock after a short delay, and asserts the call waited and then rebuilt the package whole. Inject the pause through waitBase so the row waits no real interval.
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: internal/gatecli/testmap.go:260
    summary: close is deferred before prepare and packages, so a --upkeep refusal there writes an exit record with an empty PlannedHead that never equals HEAD, which D8's restart rule reads as a reason to start another pass every tick
    fix: In close, write the exit record only once r.head is set, or move the defer run.close() write of the record after the prepare and packages refusals; keep the slot and temporary directory release unconditional.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap.go:460
    summary: A call running under a slot inherited through gateslot.InheritEnv neither takes a slot nor yields; no test pins it
    fix: In internal/gatecli/testmap_integration_test.go add a row that sets gateslot.InheritEnv to a held lease of the pool, runs a no-package call, and asserts it runs to completion without taking a second slot and without a yield.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmappackage.go:456
    summary: A terminating signal killing the runs in flight and writing the rounds that finished (D10) has no test; the existing cancel row covers a hung build only
    fix: In internal/gatecli/testmap_integration_test.go add a row that cancels the call after one round has written and asserts that round's entries are on disk and the exit code is 128 plus the signal number.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap.go:117
    summary: Under --upkeep without --max the cap is math.MaxInt and an explicit --max is honoured; every test builds the request directly and never goes through the flags
    fix: In internal/gatecli/testmap_integration_test.go add one row through the cobra command with --upkeep and no --max over more than defaultTestMapMax queued packages, asserting all are measured, and one with --upkeep --max 1 asserting the rest are queued.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap.go:246
    summary: A failed --upkeep registration (runproc.List or runproc.Register error) exits before planning; only the live-pass refusal has a row
    fix: In TestGateTestMap's Refusals add a --upkeep row whose registration directory is unwritable, asserting the refusal text, its way forward, and that no map or exit record was written.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/lymdirs/dirs.go:58
    summary: The TestMapExitFileName comment says lym gate testmap writes the file as it exits, but only a --upkeep call writes it now
    fix: Reword the comment to say a `lym gate testmap --upkeep` pass writes the file as it exits.
---

### [MEDIUM] Exit record loses unmeasured jobs on an error or signal

**Location:** internal/gatecli/testmap.go:486
**Issue:** D8 defines queued work as the work left when the pass ended.
The job loop counts the jobs left only on a HEAD move, a cap or a slotless hub.
An error return, or `r.ctx.Err()` after a terminating signal, returns with `r.queued` short by every unmeasured job.
`close` still writes `PlannedHead` = H and that `Queued`, possibly 0, so the dobby's restart rule (HEAD equal, nothing queued) starts no pass until HEAD moves.

### [MEDIUM] No test for a pass on a hub below two slots

**Location:** internal/gatecli/testmap.go:480
**Issue:** The behavior is in D9 and the Delivers line.
The only test touching slot counts covers `TryAcquireSpare` returning nil, and the only `slots: 1` row refuses earlier, at no mapped tier.

### [MEDIUM] Retake behavior after a yield is not pinned

**Location:** internal/gatecli/testmap.go:551
**Issue:** `TestGateTestMapPassYields` asserts exit 0 and the exit record's PID after the retake.
Nothing fails if the retake skips the lock retry, overwrites a map a hand call wrote meanwhile, or skips the re-plan.
The held-lock `continue` branch has no row.

### [MEDIUM] No test for the HEAD read on a slot-wait wake

**Location:** internal/gatecli/testmapyield.go:79
**Issue:** The D8 acceptance criterion names the slot-wait wake.
The HEAD-moved rows exercise the per-job read, which a broken wake read would leave passing.

### [MEDIUM] No test for a named call's wait on a held build lock

**Location:** internal/gatecli/testmapyield.go:88
**Issue:** The D9 acceptance criterion says a hand call naming a package a pass is measuring waits for its lock, does not yield and rebuilds whole.
The yield test covers a hand call that finds the lock free; `waitBuildLock` runs under no test.

### [LOW] Exit record written with an empty planned HEAD on an early refusal

**Location:** internal/gatecli/testmap.go:260
**Issue:** `defer run.close()` is installed before `prepare()` and `packages()`.
A `--upkeep` refusal there, such as an unreadable gate config, writes a record whose `PlannedHead` is empty.
It never equals HEAD, so D8's "HEAD differs from the planned HEAD" starts a fresh pass each tick.

### [LOW] Inherited-slot call has no test

**Location:** internal/gatecli/testmap.go:460
**Issue:** `spare = !named && !inheritsSlot` is the D9 line that makes such a call neither take nor yield; no row sets `gateslot.InheritEnv` in the testmap tests.

### [LOW] Signal path writing finished rounds has no test

**Location:** internal/gatecli/testmappackage.go:456
**Issue:** The existing cancel row cancels a hung build, before any round wrote.
Nothing pins that a signal after a finished round keeps that round's entries.

### [LOW] The --upkeep cap default has no test

**Location:** internal/gatecli/testmap.go:117
**Issue:** The `math.MaxInt` default lives in the cobra `RunE`, which every test bypasses by building `testMapRequest` directly.

### [LOW] Failed registration has no test

**Location:** internal/gatecli/testmap.go:246
**Issue:** D8 says a failed registration reports and exits before it plans; only the live-pass refusal is exercised.

### [LOW] Stale doc comment on the exit file name

**Location:** internal/lymdirs/dirs.go:58
**Issue:** The comment says `lym gate testmap` writes the file as it exits; since this diff only `--upkeep` does.

## Rejected

- Lens code-tests finding 8 (audit round ends on a yield or HEAD move): a branch of the yield behavior already named in F3; a second row adds no separate behavior.
- Lens code-tests finding 10 (assertions that spawn-marked and failed-at-HEAD work leave `Queued` at 0): spawn and failed-at-HEAD exclusion lies in the earlier plan card's code, which this diff does not change, and the finding names no behavior this diff adds.
- Handler check, an in-flight round whose finished runs are discarded on a yield: D9's "entries that finished" reads as finished rounds under D10's per-round write, and D4 says a killed test gets no entry; not a defect.
- Handler check, an audit round ended by a yield leaves `Queued` at 0: the audit is a closing re-measure of already-passing entries, and its mismatch count is not queued work under D8.
- Handler check, `TryAcquireSpare` counts two free slots then takes one: a gate arriving in that window leaves none free, and the yield watch returns the slot within one poll; D9 asks for a check-then-take.
