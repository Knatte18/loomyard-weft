---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loop.go:296
    summary: The blamed producer is overwritten with the after-status producer even when it is empty, so the record's fallback to the producer read before the child is not honored
    fix: In the LoopStopInterrupted branch of runLoop, assign lastKnown = after.CurrentProducer only when after.CurrentProducer is non-empty, so an empty after-read keeps the producer read before the child ran.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomengine/anomaly.go:90
    summary: ClassifyHalt reports interrupted-stop only for failed or paused, while the decision record says whatever non-blocked state the status holds
    fix: In ClassifyHalt, make the interrupted case match any state other than blocked (state != StateBlocked && interrupted), and add a TestClassifyHalt row for an interrupted halt over a running or done status; update the AnomalyInterrupted and ClassifyHalt doc comments to match.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitexec/startretry_test.go:41
    summary: No test pins the doubling backoff schedule or the retry on the remote-subcommand path, because the test sets the first delay to 0 and runs only a local subcommand
    fix: Add a row to TestRun_StartRefusedForEAGAIN that records the waits between starts through the test seam and asserts they double from the first delay, and a row that runs a remote subcommand (for example ls-remote against a local path) with refused starts and asserts the retry count.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedtransient/shedtransient.go:44
    summary: No row covers a spawn refusal older than the two-minute window, so a dropped or widened window passes every test
    fix: Add a test of spawnRefusedRecently through export_test.go (it takes now) with a refusal noted just inside and just beyond spawnRefusalWindow, asserting true then false.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loop.go:296
    summary: The empty and unreadable interrupted paths are untested - fallback to the pre-exit producer, an empty stderr file giving no trace file, and a nil StopFiles
    fix: Add subtests to TestLoop_InterruptedChildFillsTheErrorForm for a child that dies with an empty stderr and no trace file (trace_file empty) and for a status file unreadable after the child exited (hook receives the producer read before the child).
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:495
    summary: loopExitStop fills PauseReason but no test asserts its value, only the key's presence
    fix: In the loop-exit stop test, seed a paused status with a pause reason and assert the envelope's loop.pause_reason equals it.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/looparm.go:181
    summary: The loomAfterInterrupt fallback to state failed when the status file is unreadable or absent has no test row
    fix: Add a row to TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection with no status file and assert the note's state is failed with the plain cause as its reason.
  - id: F8
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/sheddriver/driverules.go:85
    summary: The paused-state guard on taking pause_reason has no row where the loop state is not paused but pause_reason is set
    fix: Add a TestParseDriveStop row with an empty reason, a non-paused loop state and a non-empty loop.pause_reason, asserting Stop.Reason stays empty.
---

### [LOW] Blamed producer can go blank

**Location:** internal/shedverbs/loop.go:296
**Issue:** `lastKnown` takes `after.CurrentProducer` whenever the status file is found. The decision record blames the status file's producer read after the child exited, falling back to the one read before it; an empty after-read yields an empty blamed row in the cause, the hook argument and the note.

### [LOW] ClassifyHalt narrower than the record

**Location:** internal/loomengine/anomaly.go:90
**Issue:** The record says `interrupted-stop` applies whatever non-blocked state the status holds. The code accepts only `failed` and `paused`. A failed write that leaves `running` (WriteFailedStop leaves any non-running state untouched and returns Busy or an error otherwise), or a child that finished the run and then died, yields an interrupted halt note with no anomaly.

### [LOW] Backoff schedule and remote path unpinned

**Location:** internal/gitexec/startretry_test.go:41
**Issue:** The first delay is 0 in the test, so `delay *= 2` cannot fail it, and only a local subcommand runs, so the retry via the remote branch of `runCoreWithin` has no coverage.

### [LOW] Spawn-refusal window expiry untested

**Location:** internal/shedtransient/shedtransient.go:44
**Issue:** `spawnRefusedRecently` takes `now`, so expiry is testable, but no row has a refusal older than `spawnRefusalWindow`. This is the record's bound that a died agent without a recent refusal stays a hard failure.

### [LOW] Interrupted-stop empty and unreadable paths untested

**Location:** internal/shedverbs/loop.go:296
**Issue:** The fallback to the pre-exit producer, an empty or missing stderr file giving no trace file, and a nil `StopFiles` have no test; the existing subtests cover only the positive cases.

### [LOW] loopExitStop pause_reason value untested

**Location:** internal/shedverbs/loopwait.go:495
**Issue:** The key-set assertions check only that `pause_reason` is present, so deleting the `PauseReason: after.PauseReason` line passes.

### [LOW] loomAfterInterrupt failed fallback untested

**Location:** internal/loomcli/looparm.go:181
**Issue:** All hook rows seed a readable status file; the default `failed` state for an unreadable or absent file has no row.

### [NIT] Paused guard on pause_reason untested

**Location:** internal/sheddriver/driverules.go:85
**Issue:** No row has a non-paused loop state with a set `pause_reason` and an empty reason, so removing the guard passes.

## Rejected

- lens:code-tests finding 2 (no `spawnagain_test.go` in `internal/proc`): the predicate, the nil case and the zero record are pinned through the public surface by `TestRun_StartRefusedForEAGAIN` and `TestClassAndMark`, so a per-helper test would duplicate them.
- lens:code-tests note on reed having no test of the shared predicate: the lens itself found no existing reed retry test to extend and the behavior is a predicate swap; not a coverage gap the diff introduces.
- lens:code-affected-set reported no findings; the handler's own holistic read of the diff against the board entry and decision record found none beyond those above.
