---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2, D4, D11
    summary: The premise that child steps die with the loop is false on Linux, so the teardown kill of D11 orphans the in-flight child, which keeps stepping the run past the session end
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2, D5, D9
    summary: No Decision covers a loop that dies without an envelope, so its waiter waits forever, the status file stays running, and a later invocation re-steps the half-run producer past its interrupt policy
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: The before and after conditions are single slots any actor can overwrite or clear, and D3 states no bound on replacing or cancelling a condition another actor set
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 and Constraints
    summary: D3 files the condition fields under shared write-to-clear, but that class lets only Shed write the cleared value, while --clear has an outside actor clear them
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1
    summary: The interrupted stop's synthesized error form omits run_id, which the closed short error key set always carries
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope, D1, D7 and Acceptance criteria
    summary: Scope and acceptance drop every empty key from the step envelope, D7 drops empty keys only inside progress, and D1 keeps the short envelopes' keys unchanged with an empty transient
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and D7
    summary: D7 says envelope_path leads to a full record naming the friction and scratch directories, but on an interrupted stop D1 points envelope_path at the loop's envelope file, which carries neither
---

### [BLOCKING] Child steps do not die with the loop

**Location:** `_lyx/discussion/decision-record.md` D2 ("Each child step inherits the loop's session, so it dies with the loop"), D4 (group kill), D11 (kill by pid file), and the acceptance row "the loop's children die with the loop".

**Issue:** `proc.Detach` on Linux is `Setsid` (`internal/proc/proc_linux.go`).
A session without a controlling terminal sends no SIGHUP to its members when the leader dies, so session membership kills nothing.
D4's group kill uses `proc.ConfigureGroupKill`, which puts each child in its own process group with `Setpgid` (`internal/proc/procgroup_linux.go`), so killing the loop's pid with `proc.KillPID` (SIGKILL to that one pid) leaves the child alone.
Windows has no session at all.
So D11's teardown kill orphans the in-flight child.
That child keeps the run lock, finishes its producer, persists, and can spawn a producer strand after the reed session has ended or while the worktree is being removed, which breaks the order PATTERN-pair-teardown guarantees.
D11's bound ("no step starts in a pair once its session-end step begins") does not cover a step already running.
For the same reason the acceptance row "the loop's children die with the loop" cannot pass as designed.

**Fix:** State a mechanism that ties the child's life to the loop's.
For example, the loop records the in-flight child's pid and start time (or pgid) beside its own, and teardown kills the child's group before or with the loop.
Or teardown first sends the loop a catchable signal on which it group-kills its child, then waits a bounded time and force-kills.
On Linux, `Pdeathsig` on the child is another option.
Name the Windows mechanism (a Job Object the loop owns).
Make D11's bound cover the in-flight step, and make the acceptance row assert that the in-flight child is dead before the reed session ends.

### [BLOCKING] A loop that dies without an envelope is unhandled

**Location:** `_lyx/discussion/decision-record.md` D2, D5 and D9 ("A job that ends without an envelope is re-run: the loop is alive and the re-run waits").

**Issue:** D2 handles a dead waiter, a live loop and a finished loop's envelope.
It does not handle a loop that dies without writing one: a crash, an OOM kill, an outside SIGKILL or a reboot.
The waiter waits for an envelope file that never arrives, and nothing bounds that wait, so the driver hangs with no stop.
That breaks the Goal's "a failed step never parks silently".
The status file stays `running`, so D5's `PATTERN-failure-stop-visible` does not hold for this failure.
A later invocation finds the loop lock free and no envelope, so it spawns a fresh loop.
That loop's first child re-invokes the producer the dead loop was running, without the `interrupt_policy` check that D4 hands to the driver for an interrupted step (Webster's `handback` exists because a re-invoke double-spawns).
D9's re-run rule assumes the loop is alive.

**Fix:** Add to D2 that a waiter seeing the loop lock free and no current envelope treats the loop as dead.
It then stops as `interrupted`, with a `loop.detail` cause such as `loop-exited`.
It writes `failed` through D5's function under the same run-lock bound, and prints the synthesized error form with the stop row's `interrupt_policy`.
It never spawns a fresh loop over a status file that still reads `running` from a dead loop.
Add an acceptance row for a loop killed mid-step.

### [MEDIUM] Overwriting or cancelling another actor's stop condition is unbounded

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** `--before` and `--after` each write one field, and both the orchestrator (through the driver, D9) and the operator may set them.
D3 does not say what a second `--before Y` does to a recorded `--before X`.
The single-field shape implies a silent replace, so one actor's stop point can vanish without notice.
`--clear` removes both conditions whoever set them.
D3 bounds `--clear` only against the bare pause request, not against a condition another actor set.
The rubric's attack-surface rule asks the introducing Decision to state what the flag can now let through and what bounds it.

**Fix:** State the rule.
For example, setting a condition over a different recorded target either refuses and names the recorded target (way forward: `--clear` first), or replaces it and reports the replaced target in the answer.
State that `--clear` removes conditions whoever set them, if that is the intent, and add an acceptance row for the chosen behavior.

### [LOW] The condition fields are filed under an ownership class whose rule they break

**Location:** `_lyx/discussion/decision-record.md` D3 ("shared write-to-clear like `pause_requested`") and the Constraints row on `Status` field ownership.

**Issue:** `internal/shedengine/status.go` defines shared write-to-clear as set true only by an outside actor and written false only by Shed.
D3 has the outside actor remove the conditions with `--clear`, which that definition forbids.

**Fix:** Name a distinct ownership class for the two condition fields: set and cleared by an outside actor through `pause`, and cleared by Shed in the paused persist and in `goto`.
State it in D3 and in the Constraints row.

### [LOW] The interrupted error form drops `run_id`

**Location:** `_lyx/discussion/decision-record.md` D1, the paragraph on the `interrupted` stop.

**Issue:** D1 says the loop's final envelope is the stop step's short envelope with unchanged keys.
The short error form is closed at `kind`, `transient`, `run_id`, `trace_file` and `envelope_path`, plus `friction` when non-empty (`internal/shedverbs/step.go`, `shortStepErrFields`).
D1's list for the synthesized form leaves out `run_id`.

**Fix:** Add `run_id` to the synthesized form, or state that the form is the short error form's closed set.

### [LOW] Three rules on dropping empty keys disagree

**Location:** `_lyx/discussion/decision-record.md` Scope ("empty keys dropped"), D1 ("unchanged keys", "`transient` is empty"), D7 ("`progress` drops `remaining` and every empty key"), and the acceptance row "The step envelope carries no `remaining` and no empty keys".

**Issue:** D7 drops empty keys only inside `progress`.
The acceptance row drops them from the whole step envelope.
Read that way, it drops an empty `transient`, `reason`, `output` or `friction`, against D1's unchanged closed key sets and its interrupted form, which carries an empty `transient`.

**Fix:** Pick one scope, most likely D7's (inside `progress` only), and word the Scope line and the acceptance row to match it.

### [LOW] `envelope_path` on an interrupted stop leads to no full record

**Location:** `_lyx/discussion/decision-record.md` D1 (`envelope_path` is the loop's envelope file) and D7 ("`envelope_path`, whose full record names the friction and scratch directories").

**Issue:** On an interrupted stop, no child wrote a step record.
`envelope_path` points at the loop's envelope file, which holds the short form plus `loop`, with no `friction_dir` or `scratch_dir`.
The fork that D9 sends to read further therefore loses the directories D7 promises.

**Fix:** Either the loop writes a full record for an interrupted stop carrying the full error envelope's location keys, and `envelope_path` names it, or D7 limits its claim to stops with a child envelope.

## Focus verdicts

- D5's `failed` write: bounded.
  The write is a Shed function, gated on a non-waiting run-lock take and a `running` re-read under that lock.
  Every other holder (Go driver, operator `step`, a second loop) makes it write nothing and stop as `busy`.
  The acceptance row "A failure stop while another holder holds the run lock leaves the status file untouched" covers it.
  Not a defect.
- D3's `--clear` against an operator's pause: bounded, since `pause_requested` is left alone.
  The ownership class is stated but mislabelled (F4).
  The conditions themselves are unbounded (F3).
- D2's liveness and spawn exclusivity: settled by the loop lock that the OS releases on death, plus a pid and start-time match for kills.
  Not a defect.
  A loop that dies without an envelope is F2.
- D2's envelope currency: settled by matching `current_producer` and `history_length` against the status file, which a `goto` or another driver's step moves.
  A `lyx loom resume` that only relaunches the driver moves neither, and delivering the stop it never saw is right.
  Not a defect.
- D11's teardown window: a defect, F1.
- D1's closed `loop` key set: it names `status_moved`, `stop`, `first_producer`, `restep` and `detail`, and `envelope_path` lives on the short form.
  A bare pause stops as `halted`.
  The interrupted error form is F5.
- D5's PATTERN entry wording: true of the paths this task changes, and it scopes out the Go driver and a bare `step`.
  F2 is a failure it does not yet cover.
