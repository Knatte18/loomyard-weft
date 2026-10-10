---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-interrupted-kind-and-arming-refusal.md
    summary: an arming refusal under --until-stop is answered by the waiter's own pre-run, before runUntilStop runs, so it carries no loop object and never writes failed, against D5, D6's acceptance row and PATTERN-failure-stop-visible
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-loop-detach-wait.md
    summary: a second waiter on a live loop misses the envelope once the first waiter retires it, then reads lock free, no envelope, spawned exited as a gone loop and reports a false loop-exited stop
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-loop-detach-wait.md
    summary: runUntilStop reads a lock held by another process's instantaneous probe as a live loop, spawns nothing, and on its next tick declares that never-existing or still-starting loop gone
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-interrupted-kind-and-arming-refusal.md
    summary: the new step arming refused row delegates its way forward to the wrapped error, but the arming errors card 9 newly routes through kind bootstrap, such as the unknown-recipe lookup, name none
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/18-pairteardown-loop.md
    summary: between killLoop and the reed session end, the still-live driver can relaunch --until-stop and spawn a detached loop that survives the session end, breaking D11's bound that no step starts once the session-end step begins
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/11-loop-body.md
    summary: card 11 does not say the --until-stop branch runs before stepCmd's in-flight record, so the waiter and the detached loop can each leave a never-finished step record that status's last_step then reports
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/12-loop-detach-wait.md
    summary: the PATTERN-failure-stop-visible overview line lacks the when-it-applies colon shape every entry line follows and internal/pattern's doc prescribes
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/18-pairteardown-loop.md
    summary: awaitRunLockFree's failure names a pid from the loop's pid file, which is not the run-lock holder when the holder is another driver or an operator step, against D11's fails with the holder's pid
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/plan/11-loop-body.md
    summary: the decision record's PATTERN-shed-verb-set constraint says the grown allowlist is stated in the entry and its background file, but cards 11 and 12 edit only the background file
---

### [BLOCKING] An arming refusal under `--until-stop` never reaches the loop

**Location:** `_lyx/plan/09-interrupted-kind-and-arming-refusal.md`, with `_lyx/plan/11-loop-body.md` and `_lyx/plan/12-loop-detach-wait.md`

**Issue:** Both step subtrees arm in their persistent pre-run, before any verb body runs (`internal/shedcli/cli.go:158`'s `resolvePersistentPreRun` calls `armFromSeed` for every verb but `shed` and `seed`; `internal/loomcli/arm.go`'s `armAt` runs the full `wire` for `step`, which is where card 9 wraps the hub landing config load).
The driver's command `lyx shed step <run-id> --until-stop` is itself a `step` invocation, so a missing hub config, an unreadable seed or any other arming error refuses in the waiter's own pre-run, through card 9's `ReportArmError`, and `runUntilStop` never runs.
Tracing the focus question: the refusal's `kind` is `bootstrap`, since the cobra verb name is still `step`.
But the stop then carries no `loop` object (D1's closed set, on which the rewritten driver stencil of card 17 branches through `loop.stop`), and nothing calls `shedengine.WriteFailedStop`, so a run whose status reads `running` stays `running`.
That is the exact `bolt-and-store` silent park D5 exists to end: D5 names "a pre-step refusal" first among the failure stops that must leave the status non-running, the acceptance row says a pre-step refusal "(missing hub config, unreadable seed) answers with `kind: bootstrap`, a trace file holding the refusal, and the status file written `failed` when one exists", and card 12's own `PATTERN-failure-stop-visible` line covers every failure stop of a `--until-stop` run.
Card 11's `stdoutErrEnvelope` path only handles a child that refuses after the waiter and the detached loop both armed cleanly, a race window, not the real case.
Card 12's `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` cannot catch this: it runs the loop in-process over `procChildRunner` with a test-built spec, so the parent's pre-run, where the real refusal fires, is never exercised.

**Fix:** Make the arming refusal of a `step` carrying `--until-stop` a loop stop: in card 9, when `ReportArmError` reports for `step` with `--until-stop` set and the run's status and lock paths resolve (the pre-run holds the location and run-id, and `internal/shedrun` constructs both), call `shedengine.WriteFailedStop` with the refusal text and its way forward, and print the error envelope with a `loop` object whose `stop` is `error` (or `busy` on a busy write), `steps` 0 and the status fields read at the stop.
Then pin it at the real surface: drive `lyx shed step <run-id> --until-stop` as a built binary over a run with the hub landing config absent, and assert `kind: bootstrap`, the `loop` key set and the status file reading `failed`.

### [MEDIUM] A second waiter loses the envelope to the first and reports a false `loop-exited`

**Location:** `_lyx/plan/12-loop-detach-wait.md`, `waitLoopEnvelope` and the retire rule

**Issue:** Card 12 has every waiter print the envelope and retire it, and ends a wait as gone when "the loop lock free, no envelope, and the spawned process, when this waiter spawned one, exited".
With two waiters on one loop (the first never killed, or a loser of a concurrent spawn whose own loop exited at `loopLockGrace`), the first to see the envelope prints and retires it.
The loop then exits and frees its lock, and it already removed its pid file after writing the envelope, so the second waiter's next tick reads lock free, no envelope, no pid file: a gone loop, reported as `interrupted` with cause `loop-exited`.
The acceptance row requires the opposite: "A second `--until-stop` while a loop is alive spawns no loop and prints the same envelope when the loop ends."
The false stop also calls `WriteFailedStop`, which is harmless only because the real stop usually left the file non-running; after a `busy` stop the file still reads `running`.
`TestUntilStop_SecondWaiterAdoptsTheLiveLoop` as planned does not pin two waiters alive at the envelope's arrival.

**Fix:** Make retirement leave a trace a concurrent waiter can still deliver, for example retire by renaming the envelope to a delivered record keyed by the loop's pid record, which a waiter that adopted that same loop prints instead of declaring it gone, and add that row (two live waiters, one envelope) to `TestUntilStop_SecondWaiterAdoptsTheLiveLoop`.

### [MEDIUM] A probe's instant hold reads as a live loop and then as a gone one

**Location:** `_lyx/plan/12-loop-detach-wait.md`, `runUntilStop`'s first read and `waitLoopEnvelope`'s gone rule

**Issue:** Every liveness probe in `internal/lock` acquires the lock: `TryAcquireWriteLock` and release, as `internal/pairteardown/teardown.go:345`'s `runLockFree` does and as card 18's quiet probe of the loop lock will.
`runUntilStop` treats "a held loop lock" as a live loop and spawns nothing.
If that read lands inside another waiter's or the teardown's probe, the invocation waits on a loop that may not exist, or on another waiter's loop still arming before it calls `AcquireWriteLockWithin`.
Having spawned nothing, its gone rule needs only lock free and no envelope, so its first tick (`waitPollFloor`, one second, shorter than a `lyx` process's arming) declares the loop gone and runs the `loop-exited` stop, writing `failed` over a run that is either idle or about to be stepped by the starting loop.
The focus asks whether any ordering lets a waiter declare a starting loop gone; this one does.
The `fswatch`-open question is fine as planned: an event only shortens the timer, so a loop killed without touching a file is still seen at the next tick, at most `waitPollCeiling` later.

**Fix:** A waiter that spawned nothing should treat a held lock as a live loop only once it has seen that loop's pid file, which the loop writes right after taking the lock; until then, or when the lock reads free with no pid file and no envelope, it re-runs `runUntilStop`'s decision, which spawns, instead of declaring a loop gone.
Add the row to `TestUntilStop_LoserWaitsOnWinner` or `TestUntilStop_DeadLoopStopsLoopExited`: the first read sees a probe-held lock and no loop exists.

### [MEDIUM] The arming refusal row's way forward is delegated to texts that name none

**Location:** `_lyx/plan/09-interrupted-kind-and-arming-refusal.md`, the `step arming refused` row

**Issue:** `PATTERN-refusal-way-forward` requires every refusal reachable through a shed or loom verb to name its way forward in its message.
Card 9 makes every arming error on `step` a `kind: bootstrap` refusal and gives it one row whose way forward is "as the wrapped error names it", but the wrapped errors mostly name none: `internal/shedcli/table.go:73`'s lookup error is `shedcli: unknown recipe %q; available recipes: %v`, the seed read's error is passed through raw, and `armAt`'s and `wire`'s errors are passed through bare.
Card 9's own test table reaches the unknown-recipe refusal on `step` and asserts only its kind and trace file.
Only the hub-config case gains a way forward.

**Fix:** Give `ReportArmError` a way-forward clause for the `bootstrap` arming refusal (for example: fix the cause the error names, then re-run the step; escalate when it is config), state that clause in the `step arming refused` row, and assert it in `TestArmFromSeed_RefusalEnvelopeKinds`'s unknown-recipe row.

### [MEDIUM] The session end leaves a window in which a new loop is spawned and outlives it

**Location:** `_lyx/plan/18-pairteardown-loop.md`, `EndSession`'s order

**Issue:** D11's bound is that no step starts in a pair once its session-end step begins.
Card 18 kills the loop, then waits up to `runLockWait` for the run lock, then ends the reed session.
In that window the driver is still alive: its waiter sees the killed loop's pid file, runs `deadLoopStop`, writes the `interrupted` envelope and retires the pid file, and the driver's interrupted rule or fork may re-launch `lyx shed step <run-id> --until-stop`, which finds no pid file and no envelope and spawns a fresh loop through `DetachBreakaway`.
That loop is in its own session, so the reed session end does not reach it, and it steps the pair after its session ended and possibly while card 18's caller removes the worktree.
The quiet probe does not close this: it runs before `killLoop`.

**Fix:** After the reed session ends and before returning, run `killLoop` once more (or end the session before the run-lock wait and kill again after it), so no loop spawned during the window survives; add the row to `teardown_test.go`'s call-sequence table.

### [LOW] The loop's own invocations may leave never-finished step records

**Location:** `_lyx/plan/11-loop-body.md`, `stepCmd`'s `--until-stop` branch

**Issue:** `stepCmd` writes the in-flight record `<trace_id>.inflight.json` under `Spec.StepsDir` before anything can refuse (`internal/shedverbs/step.go:209`), and `lastStepOf` reports the newest in-flight record as `last_step`.
Card 11 says only that "the flag runs `runLoop`"; if the branch sits after `rec.begin()`, the waiter and the detached loop each leave an in-flight record that never finishes, and a waiter re-run after the stop makes `lyx shed status`'s `last_step` report an unfinished step with the waiter's trace id, which the driver guide's interrupted and binary-change rules read.

**Fix:** State in card 11 that the `--until-stop` branch (and card 12's `--loop-detached` one) returns before the in-flight record is begun, and assert in `TestLoop_RunsUntilStopAndPrintsOneEnvelope` that the steps directory holds no record under the loop's own trace id.

### [LOW] The new PATTERN line lacks the entry shape

**Location:** `_lyx/plan/12-loop-detach-wait.md`, the `PATTERN-failure-stop-visible` line

**Issue:** `internal/pattern/doc.go` prescribes an entry line as `<when it applies>: <what it requires>.`, and every overview line follows it.
The planned line, "A `--until-stop` loop stopping on a failure with no other run-lock holder leaves the status file non-running before its waiter returns.", has no "when" clause before a colon.

**Fix:** Reword it in that shape at no byte cost, for example "Stopping a `--until-stop` loop on a failure: with no other run-lock holder, the status file is non-running before its waiter returns. (test)", and re-measure against the 16200-byte cap.

### [LOW] The run-lock wait names the wrong pid

**Location:** `_lyx/plan/18-pairteardown-loop.md`, `awaitRunLockFree`

**Issue:** D11 says the session end "fails with the holder's pid".
Card 18 fails "naming the holder's pid from the pid file", but the pid file records the loop and its in-flight child, both just killed; a run lock still held past `runLockWait` is held by someone else, a Go driver or an operator's `step`, whose pid the loop's file never names.

**Fix:** Name the run-lock holder from the run's in-flight step record (the newest `.inflight.json` under the steps directory carries the step's `pid`), or say plainly that the holder is unknown, instead of the pid file's pid.

### [LOW] The grown allowlist is not stated in the overview entry

**Location:** `_lyx/plan/11-loop-body.md` and `_lyx/plan/12-loop-detach-wait.md`

**Issue:** The decision record's constraint reads "the kind vocabulary becomes six and the allowlist grows, both stated in the entry and its background file in the same commit", and the acceptance row says PATTERN and the background files state the allowlist.
Card 9 states the six kinds in both the overview line and `pattern/PATTERN-shed-verb-set.md`, but cards 11 and 12 state the grown allowlist only in the background file.

**Fix:** Either have card 12 add a short allowlist clause to the shed-verb-set overview line within the byte cap, or, if "the entry" is read as the entry's background, say so in card 11 so the reading is explicit.

## Prior rounds

Read after this review was written.
No round-1 fix has regressed.
F1 is what remains of round 1's F2: its fix handles a child that refuses while arming, but not the waiter's own pre-run, which refuses first.
F3 is what remains of round 1's F3: the start handshake protects the waiter that spawned the loop, but not an invocation that read a probe-held lock and spawned nothing.
Round 1 deferred and disputed nothing.

## Focus departures

- The directive's card numbers do not match this plan's Card Index for most entries: it names card 10 for the loop, card 11 for `waitLoopEnvelope` and card 17 for a Windows body, while this plan's loop is card 11, the waiter card 12 and the teardown card 18.
  I followed each directive by its content, mapping it to the card that holds the named symbol or concern.
- "Cards 8 and 10, `PATTERN.md` byte budget": verdict not a defect.
  Measured with `wc -c` on today's lines and on the planned lines written out: card 8 removes 534 bytes (16375 to 15841), card 9's shed-verb-set line adds 33 (15874), and card 12's new line, background link included, adds 238 (16112), under the 16200-byte cap; every planned line is under `MaxEntryLineChars`.
- "Cards 1, 2, 11 and 17, Windows bodies": verdict not a defect.
  Cards 1, 2, 12 and 18 carry a `GOOS=windows go vet` verify over their packages, and that command type-checks `internal/proc`, `internal/gatecli`, `internal/shedverbs` and `internal/pairteardown` cleanly on this host today, so the cross-vet is runnable without cgo.
- "Cards 10, 11 and 12, trace files": verdict not a defect.
  `internal/shedverbs` reads and copies only paths told through `Spec.Loop`'s `TraceFiles` and `StopFiles`, card 11 states that in `pattern/PATTERN-shed-verb-set.md`, and card 10's `TraceFilesFor` lists by pattern because a sink file name carries a timestamp and pid that no id determines.
- "Planned tests": verdict not a defect, apart from F1's integration test, which drives `procChildRunner` in-process instead of the verb's real pre-run.
