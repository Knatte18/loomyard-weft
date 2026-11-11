---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/landing/summary.md:56
    summary: The diff leaves out two asks of issue #102 that the Goal lists, a bounded load-turn retry on server_error and the idle time in lym loom status, and no Added after Discussion entry records the omission.
    fix: Leave the code as it is and defer this finding to the run's parent, since the only sound change, recording or refusing the omission with `lym loom decision add`, is the parent's and lies outside the fixer's write surface.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loopkeeper.go:30
    summary: No test shows that the keeper survives SIGINT, SIGTERM and SIGHUP and keeps waiting; the keeper rows stub Signals with a nil channel, so deleting the handler passes every test.
    fix: In loop_unix_integration_test.go add a row to TestLoopIntegration_ProcessExitsAreMarked, or an assertion in it, that runs keepLoop with a Signals fake delivering a SIGTERM while the loop script still runs, and asserts the loop's exit mark is still written and keepLoop returns only after the loop ends.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/step.go:310
    summary: DetachedLoopID's keeper branch is untested on the arming-refusal path; the only arming test passes LoopDetachedFlag, so reverting DetachedLoopID to the detached flag alone passes every test.
    fix: In loop_arming_integration_test.go add a case to TestLoopIntegration_ArmingRefusalStopsAsBootstrap that runs the step with "--" + shedverbs.LoopKeeperFlag + "=loop-int-2" in place of the detached flag, and asserts the same bootstrap refusal arrives as the loop envelope.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loopwait.go:354
    summary: A waiter that did not spawn the keeper reports a free loop lock beside a pid record as a dead loop at once, and can read the log in the window after the loop releases its lock and before the keeper writes its exit mark, so it still reports loop exit not recorded.
    fix: In deadLoopStop, before it builds the detail from the loop log for a loop the waiter did not spawn, re-read the log once after a short bounded wait for a mark of the loop's own process exit, or have the waiter treat a live keeper as still ending the loop; keep the wait injectable through LoopSpec.Sleep so tests stay fast.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedverbs/loop.go:100
    summary: The LoopSpec.Spawn doc still says it starts the detached loop and returns a channel closed when it exits, but it now starts the keeper and the channel closes when the keeper exits.
    fix: Rewrite the Spawn field comment in loop.go so it says Spawn starts the loop's keeper with the given command line after the executable and returns a channel closed when the keeper exits; nil selects spawnLoop.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedverbs/loop.go:88
    summary: The OnStart contract says a loop that refuses before its first step never runs it, while the added line says it also runs in the keeper after the loop process starts, which happens even when that loop then refuses; doc.go:228 repeats the first claim.
    fix: In the LoopSpec.OnStart comment in loop.go and the matching sentence at doc.go:228, state that the keeper runs OnStart once after it has started the loop process, whether or not that loop then refuses, and that only the loop's own run is skipped on a refusal.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loopkeeper.go:50
    summary: Neither the keeper's start-failure path, which writes a could not start mark, skips OnStart and returns 1, nor keepLoop's return value of 0 for a clean exit and 1 otherwise has a test.
    fix: In TestLoopIntegration_ProcessExitsAreMarked add a keep row whose Executable is a path that does not exist and wants the could not start mark with OnStart not run, and make the keep helper return keepLoop's code so each keep row asserts 0 for a clean exit and 1 for the failing ones.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loopwait_test.go:522
    summary: The moved-by-child row of the dead-loop table does not assert loop.status_moved, which is now true there through hasChild and not inFlightAt, and no row covers a moved status file whose current_producer is empty.
    fix: In the StatusMovedByTheChild_StepFilesStillNamed row assert loopOf(t, env)["status_moved"] is true, and add a row, or an assertion on the ChildExited row, for a child in flight whose status file reads an empty current_producer, wanting the cause without the now-reads clause.
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loopwait.go:518
    summary: The clause and the status file now reads X also fires when X equals the in-flight producer and only the history length moved, which reads as step A was in flight and the status file now reads A.
    fix: In deadLoopStop name the status file's producer only when it differs from rec.CurrentProducer, or word the clause so it also fits a moved history length with the same producer.
---

### [MEDIUM] Two asks of issue #102 are neither made nor recorded as left out

**Location:** `_lym/landing/summary.md:56`
**Issue:** The decision record's Goal, from issue #102, asks to bound the load-turn retry on `server_error` and report the cap, and to surface the idle time between a stuck step and the next step in `lym loom status`.
The diff makes neither, and the record carries no `Added after Discussion` entry that drops them.
The summary names both as left out, so the deviation is an unrecorded parent correction.
The exit-status recording, the in-flight trace and the stderr location, which are the core of the entry, are made.

### [MEDIUM] The keeper's survival of terminating signals is untested

**Location:** `internal/shedverbs/loopkeeper.go:30`
**Issue:** The keeper's job is to outlive its sender's teardown and still write the loop's exit mark.
The `keep` rows replace `Signals` with a nil channel, and the integration test signals only the loop and kills the waiter.
Removing the handler goroutine and the signal registration would pass every test, yet teardown would then end the keeper before it wrote its mark.

### [MEDIUM] The keeper flag on the arming-refusal path is untested

**Location:** `internal/shedverbs/step.go:310`
**Issue:** `DetachedLoopID` reads the keeper flag first, and `shedcli` and `loomcli` use it in `reportArmFailure` so a refusal in the keeper's pre-run reaches the waiter as the loop envelope.
`TestLoopIntegration_ArmingRefusalStopsAsBootstrap` drives only `--loop-detached`.
Reverting the helper to the detached flag alone passes every test, and the refusal would then print in the keeper and leave the waiter reporting a vanished loop.

### [LOW] A waiter that did not spawn the keeper can still read loop exit not recorded

**Location:** `internal/shedverbs/loopwait.go:354`
**Issue:** The keeper writes the exit mark only after `cmd.Wait` returns, which is after the loop has released its lock.
A waiter with no spawn channel, such as one re-run after a driver handover, treats a free lock beside a pid record as a dead loop immediately.
In the gap it reads a log without the mark.
The window is short, and the integration test avoids it by waiting for the mark.

### [LOW] The LoopSpec.Spawn doc still describes the detached loop

**Location:** `internal/shedverbs/loop.go:100`
**Issue:** The field comment says Spawn starts the detached loop and returns a channel closed when it exits.
It now starts the keeper, and `spawnLoop`'s own comment was updated.

### [LOW] The OnStart contract contradicts itself for the keeper

**Location:** `internal/shedverbs/loop.go:88`
**Issue:** The comment says a loop that refuses before its first step never runs OnStart, and the next line says the keeper also runs it after it has started the loop process.
The keeper runs it even when the loop then refuses at the lock or the start check.
`doc.go:228` repeats the first claim without the second.

### [LOW] The keeper's start-failure path and return value have no test

**Location:** `internal/shedverbs/loopkeeper.go:50`
**Issue:** The could not start mark, the skipped `OnStart` and the exit code 1 on a failed start are untested.
The `keep` helper also discards `keepLoop`'s return value, so the 0 and 1 results are never asserted.

### [LOW] status_moved on the moved-by-child row and an empty current_producer are unasserted

**Location:** `internal/shedverbs/loopwait_test.go:522`
**Issue:** The new row gives `loop.status_moved` true through `hasChild() && !inFlightAt(after)`, and nothing asserts it.
No row covers a child in flight with a status file whose `current_producer` is empty, which suppresses the now-reads clause.

### [NIT] The now-reads clause fires for a moved history length alone

**Location:** `internal/shedverbs/loopwait.go:518`
**Issue:** When the same producer is current and only the history grew, the note reads step A was in flight and the status file now reads A.
It is not wrong, but it misleads.

## Rejected

- The keeper's own session, via `proc.Detach`, has no test that tells it from the waiter's session (lens:code-tests): the only distinguishing observation is an OS session id, and `ChildTreeDiesWithLoop` already pins the behaviour that matters, a loop that outlives the waiter's death.
- The Drift-* envelope question and the Friction-Reflect question (focus directive): the summary answers both, the child's stdout reaches the loop log only after the child exits and a normal reflection end cannot kill the loop process, and the record licenses a check without a code change, so leaving each unchanged is no defect.
- No test-economy finding is raised: the three tests the redundancy report lists are not judged by it, `ProcessExitsAreMarked` replaces the old spawn-only test with a table of distinct behaviours, and each test pins behaviour no other test covers.
