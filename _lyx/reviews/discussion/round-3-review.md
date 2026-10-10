---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D11 (line 169-171), D2 (line 76)
    summary: On Linux the parent-death signal kills only the direct child, and D11's start-time-gated group kill runs after the child is already dead, so a step's descendants outlive the loop and teardown, falsifying D11's bound
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (line 66), D11 (line 169)
    summary: proc.StartTime always reports unreadable on Windows, so a kill gated on a matching start time never fires there and D11's teardown kill of a live loop cannot pass on Windows
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (line 62)
    summary: D2 detaches the loop with proc.Detach, whose Windows form keeps the loop inside the caller's job object; the shell-survival premise is stated for Linux process groups only
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (line 67, 73-74)
    summary: The envelope currency key cannot see a resume, since the engine's resume write changes neither current_producer nor history_length, and D2 never states that a waiter retires the envelope it prints
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D2 (line 70-72), D5 (line 112)
    summary: The dead-loop stop states it writes failed and retires the pid file, but not what happens when D5's write meets another run-lock holder and stops as busy
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (line 65 vs line 70)
    summary: A waiter whose spawned loop lost the lock race and exited matches line 70's loop-exited trigger, contradicting line 65's rule that it waits on the loop holding the lock
---

### [BLOCKING] Linux: a step's descendants outlive the loop and teardown

**Location:** `_lyx/discussion/decision-record.md` D2 line 76, D11 lines 169-171; acceptance rows on lines 218, 221 and 222.

**Issue:** D2 ties the child to the loop with the mechanism `internal/gatecli/child_unix.go` and `child_linux.go` use: `Setpgid` plus `Pdeathsig = SIGKILL`.
`Pdeathsig` signals the direct child only;
the child's process group (its grandchildren, such as a verify's `lyx gate test` or a git transfer the producer started) gets nothing when the loop dies.
D11 then kills the loop first and the child's process group second, "each only while its start time matches".
Killing the loop fires the child's `Pdeathsig` at once, so by the time teardown reads the child's start time the group leader is gone, `proc.StartTime` reports unreadable (`internal/proc/proc_linux.go` reads `/proc/<pid>/stat`), and the group kill is skipped.
The run lock the child held is released with it, so D11's bounded wait passes and the worktree is removed while the step's descendants still run in it.
D11's bound "no step started before it outlives it" is therefore false for everything below the direct child, and the same gap holds for a loop that crashes on its own: D4's group kill exists only on the watchdog path.
The acceptance rows pass as written only because they name "the child", which hides the gap.

**Fix:** State in D2 and D11 a mechanism that reaches the whole step tree when the loop dies or is torn down, and its bound against pid reuse: for example teardown kills the recorded process group before the loop (gated on the loop's matching start time and the held loop lock rather than the leader's), or the group kill is gated on the recorded pgid still holding a member whose start time is at or after the recorded child start.
Extend the teardown and loop-killed acceptance rows to assert that a grandchild of the step is dead.

### [BLOCKING] Windows: the start-time gate never matches, so teardown never kills the loop

**Location:** `_lyx/discussion/decision-record.md` D2 line 66, D11 line 169; acceptance row on line 221.

**Issue:** D2 and D11 allow a kill "only while the loop lock is held and the start time still matches".
`internal/proc/proc_windows.go`'s `StartTime` returns `"", false` unconditionally ("keeps the resume check failing closed there").
On Windows the pid file can record no start time and no later read can match one, so D11's session-end kill of a live loop never acts;
the bounded wait for the run lock then fails with the holder's pid, and a pair with a live loop cannot be torn down on Windows until the loop stops on its own.
The repo targets Windows (D2 already names the Job Object arm), so the teardown acceptance row cannot pass there as designed.

**Fix:** Name in D2 and D11 the Windows identity check the kill uses in place of the start time (for example a process creation time read through `GetProcessTimes`, added to `internal/proc`), or a different Windows kill route that does not need one, such as a named job object the teardown opens and terminates.
State the bound it gives against a reused pid.

### [MEDIUM] Windows: the detached loop's survival of the driver's shell is unstated

**Location:** `_lyx/discussion/decision-record.md` D2 line 62.

**Issue:** D2's premise is that the driver's shell is killed "with its process group", and `proc.Detach` (`Setsid` on Linux) answers that.
On Windows `proc.Detach` sets only `CREATE_NO_WINDOW | CREATE_NEW_PROCESS_GROUP`;
a process spawned that way stays in its parent's job object, and the repo already has `proc.DetachBreakaway`, documented as "additionally surviving a Windows Job Object", for exactly that case.
The cited precedent, `lyx loom start`'s Go arm, is spawned from an operator's shell, not from inside an agent's background job, so it does not establish that `Detach` suffices here.
If the driver's shell on Windows is torn down through a job, the loop dies with it and D2's purpose fails there.

**Fix:** State in D2 which detach form the loop uses on Windows and why it survives the driver's shell there, choosing `DetachBreakaway` if the shell can sit in a kill-on-close job.

### [MEDIUM] The envelope currency key cannot see a resume, and watched-envelope retirement is unstated

**Location:** `_lyx/discussion/decision-record.md` D2 lines 67, 73-74.

**Issue:** D2 delivers an unwatched loop's envelope only while its `loop.current_producer` and `loop.history_length` match the status file, and calls an envelope stale when "the run has moved past [it] by a resume".
The engine's resume write (`internal/shedengine/run.go`, step 3b) writes `running` with the same `current_producer` and the same history, and a halt writes no history either (D5's `failed` write is "history untouched"), so a resume never changes either key.
`appendOrFold` also folds a repeated budget-exempt Stuck at the same producer into the last entry, so `history_length` is not a strict move counter.
Concretely: a loop ends unwatched at `awaiting` or `paused` while its driver session is dead (the case the Open risks name), the operator approves and resumes, and the new driver's `--until-stop` is handed the old halt envelope as current, so it hands back a stop that is already resolved.
Separately, D2 says only that an unwatched envelope is retired after delivery; an invocation cannot tell watched from unwatched, so unless a waiter retires the envelope it prints, every later invocation re-delivers the last stop while the keys match, which after an error stop with an unchanged status is every re-launch.

**Fix:** State in D2 that a waiter retires the envelope it prints, and make the currency check one a resume changes: include the status file's `state` (an envelope recorded at a halt is stale once the file reads `running`), or stamp the envelope with a stop serial the engine's persist advances.
Correct the "moved past by a resume" premise to match.

### [MEDIUM] The dead-loop stop under a busy run lock leaves the pid file's fate open

**Location:** `_lyx/discussion/decision-record.md` D2 lines 70-72, D5 line 112.

**Issue:** D2 says the dead-loop stop "writes `failed` through D5's function under D5's bound, writes its envelope as the loop's envelope file, and retires the pid file", and its bound rests on the pid file: "an invocation never spawns a loop over a dead loop's pid file".
D5's bound makes the write a no-op and the stop `busy` when another holder has the run lock.
D2 does not say whether a `busy` dead-loop stop still retires the pid file and writes an envelope.
If it does, the next invocation finds no pid file, spawns a fresh loop over a status file that may still read `running` for the half-run producer, and re-invokes it without the `interrupt_policy` check, the exact case the bound exists to prevent.
If it does not, the next invocation retries the dead-loop stop, which is the safe reading but is not stated.

**Fix:** State in D2 that a dead-loop stop that ends `busy` keeps the pid file and writes no loop envelope, so the next invocation retries it, and add the case to the loop-killed acceptance row.

### [LOW] A lock-race loser's exit matches the loop-exited trigger

**Location:** `_lyx/discussion/decision-record.md` D2 line 65 vs line 70.

**Issue:** Line 65 says a loop that cannot take the loop lock exits at once without an envelope, and "its waiter waits on the loop that holds the lock".
Line 70 says "a waiter whose loop is gone (its spawned loop exited, or the lock it watched was released)" stops the run as `interrupted`, `loop-exited`.
The loser's waiter satisfies line 70's first clause literally, and it would then try to write `failed` while the winner is live.
D5's bound catches it only while the winner's child holds the run lock;
between two children the run lock is free and the status file reads `running`, which is the ordinary between-step state, so D5's guard alone does not tell a live loop from a dead one.

**Fix:** Define "the loop is gone" in D2 by the loop lock alone: a waiter treats its spawned loop's exit as gone only when the loop lock is then free and no envelope exists, and otherwise waits on the lock's holder.

## Focus verdicts

- Focus 1 (child lifetime and D11's bound): the direct child is tied to the loop on both platforms by a stated mechanism, including the locked-thread requirement; the step's descendants are not on Linux (F1), and the teardown kill cannot act on Windows (F2). The acceptance rows on lines 218 and 222 pass as written for the direct child; the teardown row on line 221 cannot pass on Windows.
- Focus 2 (lock free, no current envelope): with a dead loop's pid file present, D2 stops as `interrupted` and never spawns, so the `interrupt_policy` check is kept; the gaps are the `busy` branch (F5), the loser-exit wording (F6) and envelope currency (F4).
- Focus 3 (`--before`, `--after`, `--clear` over another actor's condition): not a defect. D3 lines 89-90 state the one-slot replace rule and the overriding answer, line 86 bounds `--clear` away from the bare request, and the acceptance row on line 210 covers replacement, the answer, and `--clear` leaving a bare request.
- Focus 4 (ownership class): not a defect. `internal/shedengine/status.go` splits ownership three ways, with `PauseRequested` set only by an outside actor and cleared only by Shed; D3's fourth class, set and cleared by an outside actor through `pause` and cleared by Shed only at the firing persist and a `goto`, matches who writes the fields, and the Constraints row on line 185 names the same class.
- Focus 5 (empty keys, interrupted error form): not a defect. Scope line 22, D7 line 133 and the acceptance row on line 219 all scope the empty-key drop to the step envelope's `progress`. D1's synthesized form (`error`, `kind`, `transient`, `run_id`, `trace_file`, `envelope_path`, `friction` when written) equals `shortStepErrFields`' closed set in `internal/shedverbs/step.go`, and the full record's location keys (`trace_file`, `friction_dir`, `scratch_dir`, `trace_id`, `run_id`) cover the friction and scratch directories D7 promises.
