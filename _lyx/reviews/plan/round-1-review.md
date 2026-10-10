---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-loop-body.md:64
    summary: The PATTERN.md overview ends at about 16382 bytes after card 10, over the 16200-byte cap the decision record sets; card 8's six rewordings free only about 238 bytes
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-loop-body.md:52
    summary: A pre-step arming refusal writes no step record, so the loop classifies it as interrupted with cause exited instead of the bootstrap error D6 and the acceptance criteria require
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/plan/11-loop-detach-wait.md:46
    summary: The loop lock as liveness record races at loop start, because a waiter declares the loop gone before the spawned loop takes the lock, and every try-acquire probe can make a starting loop exit
  - id: F4
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/10-loop-body.md:56
    summary: Cards 10 and 12 join, read and copy files under logger.TraceDir inside internal/shedverbs, which PATTERN-shed-verb-set forbids, and trace file names carry a timestamp and pid that no join can derive
  - id: F5
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-loop-body.md:56
    summary: The interrupted and loop-exited stops get refusal-spec rows, but their error text is the bare cause with no way forward, against PATTERN-refusal-way-forward
  - id: F6
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-pause-conditions.md:21
    summary: Several cards plan tests that drive unexported helpers (applyPauseConditions, reportArmError, runLoop, runUntilStop) rather than the verb surface, against PATTERN-test-economy
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-loop-detach-wait.md:46
    summary: The waiter wakes on fswatch events alone where fswatch opens, and a loop that dies without writing a file produces no event, so the waiter never sees the lock go free
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-loop-detach-wait.md:42
    summary: The hidden loop-detached flag is a new escape hatch with no stated bound, and runLoop entered through it writes over a dead loop's pid file, against D2's bound
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/plan/01-proc-tied-child.md:37
    summary: No command in the plan compiles the new Windows bodies, and card 1's claim that go vet compiles them here is false on a Linux host
  - id: F10
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/19-stencil-drift-once.md:14
    summary: Card 19 changes what decide returns but has no re-sign arrow, and leaves out the decision struct, reconcileOne and pendingWrites, through which the notices must flow
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/08-interrupted-kind-and-arming-refusal.md:43
    summary: The kind-less-on-step row of TestReportArmError covers the behavior TestArmFromSeed_MissingRunRefusalCarriesNoKindField already pins, and that test keeps bypassing the new render path
  - id: F12
    severity: LOW
    class: design
    location: _lyx/plan/12-loop-watchdog.md:25
    summary: The watchdog sends SIGQUIT and then SIGKILL at once, with no bounded grace, so the goroutine dump D4 wants in the stderr file can be cut off
  - id: F13
    severity: LOW
    class: design
    location: _lyx/plan/11-loop-detach-wait.md:47
    summary: A loop that dies after writing its envelope but before removing its pid file leaves a pid file that, once the waiter retires the envelope, reads as a dead loop and triggers a false loop-exited stop
  - id: F14
    severity: LOW
    class: scope
    location: _lyx/plan/12-loop-watchdog.md:25
    summary: D2 requires the loop to log each kill, but no card states logging for the watchdog kill, the dead-loop tree kill or the teardown kills
  - id: F15
    severity: LOW
    class: consistency
    location: _lyx/plan/18-plan-gate-findings-file.md:16
    summary: Card 18 writes the findings file to webster's scratch directory, not the run's scratch directory that D10 names
  - id: F16
    severity: LOW
    class: consistency
    location: _lyx/plan/01-proc-tied-child.md:28
    summary: On Windows KillTree terminates the recorded job by name, but the per-child jobs D2 describes are unnamed, so the Windows half of TreeRecord and KillTree disagree
  - id: F17
    severity: LOW
    class: design
    location: _lyx/plan/08-interrupted-kind-and-arming-refusal.md:48
    summary: Card 8 bundles three independently reviewable units, namely the sixth kind, the D6 arming refusal and six unrelated PATTERN overview rewordings
  - id: F18
    severity: LOW
    class: consistency
    location: _lyx/plan/10-loop-body.md:64
    summary: Card 10 lands PATTERN-failure-stop-visible, which states a guarantee about the waiter and covers the dead-loop path, but card 11 builds both
  - id: F19
    severity: NIT
    class: scope
    location: _lyx/plan/15-start-keeps-status-strand.md:17
    summary: Card 15 leaves the start.go file header, which still says the status strand is removed on an llm-driven run
---

### [BLOCKING] The PATTERN overview breaks its cap after card 10

**Location:** `_lyx/plan/10-loop-body.md:64`, `_lyx/plan/00-overview.md:37`, `_lyx/plan/08-interrupted-kind-and-arming-refusal.md:48`

**Issue:** `PATTERN.md` is 16375 bytes now.
I measured card 8's seven rewritten lines as they would be written, each with the background link that a new `pattern/` file needs (every file under `pattern/` must be named by an entry, `internal/pattern/doc.go`).
Together they come to 1723 bytes against 1961 today, a saving of about 238 bytes, which leaves the file at about 16137 after card 8.
Card 10's new `PATTERN-failure-stop-visible` line, with its background link, is about 245 bytes, which brings the file to about 16382.
That is over the decision record's 16200-byte constraint and 2 bytes under the hard 16384 cap.
The Shared Decision `pattern-overview-cap` says card 8 pays for card 10's line, but the numbers show it does not.

**Fix:** Have card 8 shorten more overview wording by at least about 190 bytes beyond what it plans now, moving detail into background files as the constraint allows, and shorten card 10's new line.
State in the Shared Decision the byte count each card leaves, so the fixer can check it.

### [BLOCKING] An arming refusal reaches the loop as interrupted, not bootstrap

**Location:** `_lyx/plan/10-loop-body.md:52`, `_lyx/plan/08-interrupted-kind-and-arming-refusal.md:40`

**Issue:** Card 10's loop reads only the child's step record, `<StepsDir>/<traceID>.json`, and treats "a child that left no record" as `interrupted`.
That record is written by `stepCmd`'s RunE through `newStepRecorder(spec.StepsDir, …)` (`internal/shedverbs/step.go:210`, `steprecord.go`).
Card 8's `reportArmError` runs in `internal/shedcli`'s `resolvePersistentPreRun`, before any spec exists and before `stepCmd` runs, so it prints an envelope on stdout and writes no record.
Card 11's own integration test confirms the mechanism: an unseeded child "exits without a record and stops the loop as `interrupted` with cause `exited`".
Under `--until-stop`, which is the driver's only path after card 16, a missing-hub-config refusal therefore answers `kind: interrupted`, not D6's `kind: bootstrap`, which fails the acceptance row "A pre-step refusal … answers with `kind: bootstrap`".
The `lyx loom step` subtree has its own pre-run in `internal/loomcli`, and card 8 does not touch it, so the loom subtree's arming refusals stay kind-less there too.

**Fix:** Either have card 8's `reportArmError` write the step record under `shedrun.StepsDir` for the resolved run, so the loop reads it as an error stop (shedcli already resolves through `shedrun` constructors), or have card 10 parse the child's stdout envelope when no record exists.
Cover the loom subtree's pre-run as well, or state why D6 binds only `lyx shed step`.
Add a loop test row in which a child's arming refusal stops the loop as `error` with kind `bootstrap`.

### [BLOCKING] The loop-lock liveness check races at loop start

**Location:** `_lyx/plan/11-loop-detach-wait.md:46`, `_lyx/plan/11-loop-detach-wait.md:43`, `_lyx/plan/17-pairteardown-loop.md:31`

**Issue:** `waitLoopEnvelope` ends "when the loop lock is free with no envelope (the loop is gone)" and then reports `interrupted`/`loop-exited`.
Right after `spawnLoop` returns, the detached process has not yet started or taken the lock, so a waiter that checks at once sees a free lock and no envelope, declares the loop dead and may run `WriteFailedStop` on a running run.
"Held" is read the way `pairteardown.runLockFree` reads it, by `lock.TryAcquireWriteLock` and then release (`internal/pairteardown/teardown.go:345`).
While a probe holds the lock, a starting loop's own `TryAcquireWriteLock` fails, and card 11 makes that loop "exit at once … without an envelope".
Every concurrent probe can therefore kill a starting loop: the first check in `runUntilStop`, a second waiter, and card 17's `probeQuiet`.
D2's "a waiter whose spawned loop exited while the lock is held waits on the holder" assumes the holder is a loop, not a probe.

**Fix:** Have card 11 state a start handshake: the waiter treats a spawned pid that is still alive, or a pid file not yet written, as "starting", and decides "gone" only once the spawned process has exited and the lock is free with no envelope.
Probing liveness without taking the lock, or letting a loop retry its lock acquire for a bounded few seconds, removes the steal.
Add a `loopwait_test.go` row for a waiter that polls before the loop takes the lock.

### [BLOCKING] `shedverbs` joins and reads trace paths, which PATTERN-shed-verb-set forbids

**Location:** `_lyx/plan/10-loop-body.md:56`, `_lyx/plan/10-loop-body.md:59`, `_lyx/plan/12-loop-watchdog.md:23`

**Issue:** `pattern/PATTERN-shed-verb-set.md` admits `logger.TraceFile` and `logger.TraceDir` into `shedverbs` only on the condition that `shedverbs` "reports verbatim on the envelope and never joins, reads, writes or uses to locate run state".
Card 10 sets the interrupted envelope's `trace_file` to "`logger.TraceDir()` joined with the trace file name for that id", reads the stop trace's WARN/ERROR lines and copies it.
Card 12's `newestActivity` reads the mtime of the child's trace group under `logger.TraceDir()`.
The join also cannot be built: the sink names files `trace-<timestamp>-<traceID>-<pid>.log` (`internal/logger/sink.go:135`), so the loop cannot spell a child's name from its id, and a trace group may span several files.
Neither card's edit to `PATTERN-shed-verb-set.md` states a new admission.

**Fix:** Tell the trace reading through `LoopSpec`, as the loop paths are already told.
One way is a closure filled by the arming module that maps a trace id to its trace files, backed by an exported `logger` lookup that card 9 adds beside `NewTraceID`; fold the trace mtime into the `Activity` hook card 14 fills.
Alternatively, amend the entry's background in card 10 to admit trace reads by id, which changes an invariant and must say so.

### [BLOCKING] The interrupted stops have no way forward in their message

**Location:** `_lyx/plan/10-loop-body.md:56`, `_lyx/plan/11-loop-detach-wait.md:51`

**Issue:** Card 10 adds the `step interrupted` refusal row, and card 11 adds the `loop exited` row, both with "way forward: the driver's interrupted rule".
The envelope's error, and the text `WriteFailedStop` writes into the status file's `error`, is "the cause" (`watchdog`, `exited`, `loop-exited`, `status-missing`) with no way-forward clause.
`PATTERN-refusal-way-forward` requires every refusal of shed and loom to name its way forward in its message.
D5 also says the failed write carries "the error text and the way forward", and card 5's `FailedStopRequest` has a way-forward field that card 10 never fills for these stops.
Card 6's contradictory `--clear` refusal likewise gets a row but no stated message or way forward.

**Fix:** Have cards 10 and 11 compose each interrupted cause into an error ending in `way forward: …`, for example re-run `lyx shed step <run-id> --until-stop` after reading `loop.trace_copy`, and pass that clause to `WriteFailedStop`.
Have card 6 state the contradictory-flag message with its way forward.
The loop tests then assert the clause.

### [BLOCKING] Planned tests drive unexported helpers instead of the verb surface

**Location:** `_lyx/plan/06-pause-conditions.md:21`, `_lyx/plan/08-interrupted-kind-and-arming-refusal.md:43`, `_lyx/plan/10-loop-body.md:68`, `_lyx/plan/11-loop-detach-wait.md:58`

**Issue:** `pattern/PATTERN-test-economy.md`: "an unexported helper is tested through the surface that uses it."
Card 6 makes `applyPauseConditions` "a pure function the test drives directly", yet the same test claims to assert "leaving the file unchanged", which only the verb can show.
Card 8's `TestReportArmError` drives the unexported `reportArmError`.
Card 10's tests drive `runLoop`, and card 11's tests drive `runUntilStop` and `runLoop` through a fake `Spawn`.
The package's own surface test idiom already exists: `execEnvelope(t, pauseCmd(...), …)` in `internal/shedverbs/pause_test.go`.
Every loop seam card 10 adds (`Runner`, `Spawn`, `Watch`, `Now`, `Sleep`) sits on the `Spec` that `stepCmd` takes, so the same tests can run through `stepCmd(texts, spec)` with `--until-stop`.
This is one method gap repeated across cards, so it is filed once.

**Fix:** Have card 6 assert through `pauseCmd` with `execEnvelope` and the re-read status file, card 8 through `resolvePersistentPreRun` on a `step` command, and cards 10 to 12 through `stepCmd` with `--until-stop` over the faked `LoopSpec`.

### [MEDIUM] A dead loop is invisible to an fswatch-only waiter

**Location:** `_lyx/plan/11-loop-detach-wait.md:46`

**Issue:** `waitLoopEnvelope` waits "through `fswatch.Watch` on the steps directory where it opens, else by polling".
A loop that is OOM-killed, killed from outside or crashes releases its `flock` without touching a file, so no event fires and a waiter woken only by events never reaches its "lock free with no envelope" check.
The driver's background job then hangs until the shell's time limit, which is the failure D2 exists to remove.
Card 11's tier-1 test fakes `Watch` and would not show it.

**Fix:** State that the waiter also re-checks the loop lock on the backoff timer (floor 1s, ceiling `waitPollCeiling`) while fswatch is open, with events only shortening the wait.
Add a `TestUntilStop_DeadLoopStopsLoopExited` row in which the fake watch emits nothing.

### [MEDIUM] The hidden `--loop-detached` flag has no stated bound

**Location:** `_lyx/plan/11-loop-detach-wait.md:42`, `_lyx/plan/11-loop-detach-wait.md:48`

**Issue:** The flag is a new route into `runLoop` that skips `runUntilStop`'s checks.
`runLoop` takes the loop lock and writes the pid file "after taking the lock", and nothing in it looks for a dead loop's pid file.
`lyx shed step <run-id> --until-stop --loop-detached` over a dead loop therefore overwrites the record of the dead child, so `deadLoopStop` never kills its tree, and it re-steps the half-run producer.
D2's bound "an invocation never spawns a loop over a dead loop's pid file" is broken by that route.
The rubric's attack-surface rule requires the Intent to state what a new flag lets through and what bounds it.

**Fix:** Have `runLoop` refuse to start, with a way forward, when it holds the lock and finds a pid file and no envelope.
State in card 11's Intent that the flag only marks the spawned process and is bounded by that check.

### [MEDIUM] Nothing in the plan compiles the new Windows code

**Location:** `_lyx/plan/01-proc-tied-child.md:37`, `_lyx/plan/00-overview.md:72`

**Issue:** Card 1 says "Windows bodies compile under `go vet` here".
`go vet ./...` on a Linux host skips `_windows.go` files, and no file in the repository runs a `GOOS=windows` build or vet.
Cards 1, 2, 11 and 17 add named jobs, `TerminateJobObject`, breakaway detection and the Windows arms of the gate child and teardown, and none of that is type-checked before landing.

**Fix:** Add a package-scoped `**Verify:**` to card 1, for example `GOOS=windows go vet ./internal/proc ./internal/gatecli`, and extend it on cards 11 and 17 to `./internal/shedverbs ./internal/pairteardown`.
Correct card 1's sentence.
A per-card Verify is admissible because a package-scoped automatic run cannot catch this.

### [MEDIUM] Card 19 re-signs `decide` with no arrow and misses its callers

**Location:** `_lyx/plan/19-stencil-drift-once.md:14`

**Issue:** "`decide` returns its refresh notices … to `reconcileAll`" changes what `decide` returns, either its results or the `decision` struct (`internal/stencilstore/reconcile.go:176`).
`reconcileAll` does not call `decide` directly.
`reconcileOne` (`reconcile.go:257`) and `pendingWrites` (`reconcile.go:150`, the `WritesDue` path) do, so the notices must flow through `reconcileOne`'s results.
`pendingWrites` would otherwise drop them or emit them twice.
The card lists none of `decision`, `reconcileOne` or `pendingWrites`, and `decide` carries no re-sign arrow.

**Fix:** List `internal/stencilstore#decision` (field change, no arrow) or give `decide` its re-sign arrow.
Re-sign `reconcileOne` with its arrow, list `pendingWrites`, and state what the `WritesDue` path does with the notices.

### [LOW] TestReportArmError overlaps an existing test

**Location:** `_lyx/plan/08-interrupted-kind-and-arming-refusal.md:43`

**Issue:** The row "a kind-less refusal on `step` prints no kind" covers what `TestArmFromSeed_MissingRunRefusalCarriesNoKindField` (`internal/shedcli/cli_test.go:50`) pins, the missing-run refusal's two-key envelope.
That test renders through `output.Err` itself, so after card 8 it pins nothing on the real render path.

**Fix:** Fold the new rows into that test, renamed to cover arming refusal envelopes, and render through the pre-run instead of calling `output.Err` directly (see the test-surface finding above).

### [LOW] SIGQUIT has no grace before SIGKILL

**Location:** `_lyx/plan/12-loop-watchdog.md:25`

**Issue:** `runLoop` "calls `Child.Quit` … then `Child.Kill`" with no wait between them.
A Go runtime needs time to write its goroutine dump, and an immediate group SIGKILL can cut it off, losing the stderr evidence D4 asks for.

**Fix:** State a bounded grace: wait for the child to exit for a few seconds after `Quit`, then `Kill`.
Keep the grace injectable so the fake-clock test does not wait it out.

### [LOW] A leftover pid file after a delivered envelope reads as a dead loop

**Location:** `_lyx/plan/11-loop-detach-wait.md:47`

**Issue:** The loop removes its pid file only after the envelope is on disk.
If it dies in between, a waiter delivers and retires the envelope while the pid file stays.
The next `--until-stop` then sees a free lock, a pid file and no envelope, and runs `deadLoopStop`, which writes `failed` over a run that has since been resumed and reports a stop that never happened.

**Fix:** Have the waiter, and the unwatched-envelope path, retire a pid file it finds beside a delivered envelope while the loop lock is free.

### [LOW] No card states that the kills are logged

**Location:** `_lyx/plan/12-loop-watchdog.md:25`, `_lyx/plan/11-loop-detach-wait.md:51`, `_lyx/plan/17-pairteardown-loop.md:32`

**Issue:** D2: "The loop logs each child spawn and each kill through `internal/logger`."
Card 10 logs spawns and stops, and card 11 logs the loop spawn.
No card states logging for the watchdog's `Quit`/`Kill`, `deadLoopStop`'s `KillTree` or card 17's loop and tree kills.

**Fix:** Have each of those cards' Intent state the kill's log line with the pid or group and the outcome.

### [LOW] The findings file lands outside the run's scratch directory

**Location:** `_lyx/plan/18-plan-gate-findings-file.md:16`

**Issue:** D10 places the file "under the run's scratch directory".
Card 18 uses `websterengine.ScratchDir(anchorPath)`, which is `.lyx/webster/` (`internal/websterengine/state.go:65`), webster's scratch directory and not the run's `shedrun.ScratchDir(loc, runID)`.

**Fix:** Tell the gate the run's scratch directory through `NewPlanGate`/`NewReworkPlanGate`'s wiring, or state why webster's directory stands in for it.

### [LOW] Card 1's Windows tree kill names a job that has no name

**Location:** `_lyx/plan/01-proc-tied-child.md:28`

**Issue:** `TreeRecord` carries "on Windows, the per-child job handle's identity", and `KillTree` on Windows "terminates the recorded job by name through `TerminateNamedJob`".
D2's per-child jobs are unnamed kill-on-close jobs, and a handle identity cannot be reopened from another process.

**Fix:** State that `KillTree` on Windows reports nothing killed, because the per-child job died with its loop, or name the per-child jobs.
Drop the job identity from `TreeRecord` if nothing reads it.

### [LOW] Card 8 bundles three units

**Location:** `_lyx/plan/08-interrupted-kind-and-arming-refusal.md:48`

**Issue:** The decision record's overview-cap constraint licenses each of the six rewordings, and none drops an invariant.
They are still a doc-only unit unrelated to the arming refusal and the kind vocabulary, and `KindInterrupted` has no emitter before card 10.
Splitting them into their own card ahead of card 8 gives a reviewable cap-making card, and that card is where the cap fix above belongs.

**Fix:** Move the six rewordings and their new background files to their own card before card 8.

### [LOW] Card 10 lands an invariant that card 11 completes

**Location:** `_lyx/plan/10-loop-body.md:64`

**Issue:** The new entry guarantees non-running "before the waiter returns", and its scope includes a loop that died without an envelope (D5).
Card 10 has no waiter and no dead-loop stop; both arrive in card 11.
Otherwise card 10 is a sound unit: its docs, refusal rows and budget belong in the same commit as the code they describe.

**Fix:** Move the `PATTERN.md` line and `pattern/PATTERN-failure-stop-visible.md` to card 11.

### [NIT] The start.go header still says the strand is removed

**Location:** `_lyx/plan/15-start-keeps-status-strand.md:17`

**Issue:** `internal/loomcli/start.go:2` says the status strand is "kept on a go-driven run, removed on an llm-driven one", and the card's Intent does not mention that comment.

**Fix:** Name the file-header rewording in the Intent.

## Focus questions found sound

Cards 10 to 12 state bounds for `--until-stop`, the Detach fallback, the dead-loop stop and the watchdog kill (the hidden flag is the exception, see above).
Cards 4 and 6 state D3's `--clear` and retarget bounds.
Card order has no forward dependency.
`## verify:` covers every targeted package in the template's order (the Windows gap is above).
`armFromSeed`, `resolvePersistentPreRun`, `stepCmd`, `wire` and `specFor` keep their signatures, so they need no arrow.
