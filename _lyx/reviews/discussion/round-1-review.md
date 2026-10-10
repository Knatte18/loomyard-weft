---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D5)
    summary: The loop's failed write is bounded only by the file still reading running under the status lock, which neither proves the loop owns the run nor respects the engine's sole ownership of State, and no acceptance criterion reaches a run another holder owns.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3)
    summary: pause --clear also clears the bare pause request, which breaks the PauseRequested write-to-clear contract and lets any caller cancel an operator's pause, and D3 states no bound and no ownership class for the two new condition fields.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2)
    summary: Pid-file liveness has no bound against a stale or reused pid, and nothing makes the spawn exclusive when two waiters both find no live loop.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2)
    summary: An envelope from a loop that ended unwatched is delivered by the next invocation with no staleness bound, so a stop the run has since moved past (resume, goto, Go driver steps) reaches the driver as current.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D1, D4, D7, D8, Acceptance criteria)
    summary: The closed loop key set in D1 disagrees with what D4, D7, D8 and the acceptance criteria put on the envelope, and the stop cause for a bare pause and the step keys of an interrupted stop are undefined.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D11)
    summary: Teardown kills the loop only after ending the reed session, so a live loop can step the run and spawn producer strands in a pair being torn down.
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D5, Scope Out)
    summary: PATTERN-failure-stop-visible is worded for every driven run, but only the until-stop loop implements it, and the Go driver lyx loom run is out of scope.
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md (D3)
    summary: The lifetime of a condition whose target is never reached again, or that a goto skips past, is left open, and goto already clears the bare pause request.
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md (D8)
    summary: D8 leaves open whether a second failure at the same producer means two consecutive failures or any two in the loop's lifetime.
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Added after Discussion)
    summary: The PATTERN overview cap entry is a card-level constraint filed as an unnumbered Decision, it pins the current byte count, and its run history is deliberation narrative.
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (D10)
    summary: D10 restates D4's routing of the SIGQUIT dump and the child's stderr to the captured stderr file.
---

### [BLOCKING] The loop's `failed` write has no ownership bound

**Location:** `_lyx/discussion/decision-record.md`, D5; Acceptance criteria, the interrupted and pre-step rows.

**Issue:** D5 lets the loop write `failed` whenever, after a stop, the status file "still reads `running`", taking only the status lock.
The bound does not settle the focus question about "a status file another holder may own".
`running` does not mean the loop's child owns the run.
After a child dies or is killed, its run lock is released, and a Go driver `lyx loom run`, an operator's `lyx shed step` or a second loop (see F3) can take the run and be mid-producer, with the file reading `running` on their behalf.
The loop then writes `failed` over a live step.
D5 exempts only a *busy refusal*, the case where the loop itself was refused.
It does not cover a holder that takes the run after the loop's child is gone.
Separately, `internal/shedengine/status.go`'s `Status` doc gives `State` and `Error` to Shed alone ("Shed-owned and rewritten on every persist").
D5 has the loop write them from `internal/shedverbs`, outside the engine.
That is a guard change D5 does not name, and it is not on the Constraints list either.
No acceptance criterion covers a run that another holder owns at the moment of the write.

**Fix:** In D5, route the write through one `shedengine` function that writes `failed` only while it holds the run lock that the busy refusal reads.
If the lock is held, the function writes nothing and the loop stops as busy.
Then the engine's ownership of `State` holds and the bound is concrete.
Add an acceptance row: a failure stop while another holder holds the run lock leaves the status file untouched.
Add the change to the `Status` ownership contract to the Constraints line for docs.

### [MEDIUM] `--clear` removes the bare pause request without a bound

**Location:** `_lyx/discussion/decision-record.md`, D3; Acceptance criteria, the stop-condition row.

**Issue:** D3's `--clear` "removes both conditions and the bare pause request".
`internal/shedengine/status.go` documents `PauseRequested` as "set true only by an outside actor and written false only by Shed, exactly once, in the persist that records StatePaused".
`--clear` is a second writer of `false` and breaks that contract.
It also lets any caller of `--clear`, whether the driver acting on an orch message or the orch itself, cancel a pause the operator requested, so a guard becomes bypassable.
D3 does not say what `--clear` can now let through or what bounds it.
The acceptance row checks that `--clear` removes the request, but nothing checks the bound.
D3 also gives no ownership class for the two new condition fields.
It says only that the engine consumes them "where it consumes `pause_requested`".

**Fix:** Either `--clear` removes only the two conditions and leaves `pause_requested` alone, which is the narrower edge, or D3 states the bound and amends the `Status` ownership contract in the same commit.
The bound would say who may clear an operator's pause, and that it is the same actors who may set one.
Either way, D3 states that the two condition fields are shared write-to-clear like `PauseRequested`, and the Constraints docs line names the `Status` contract change.

### [MEDIUM] Pid-file liveness and spawn exclusivity are unbounded

**Location:** `_lyx/discussion/decision-record.md`, D2.

**Issue:** D2 says "the loop's pid file in the steps directory is the liveness record", and a second `--until-stop` "while the loop is alive" waits instead of spawning.
A pid file left by a loop that died without cleanup, such as after a SIGKILL or a reboot, can name a pid the OS has reused.
The waiter then waits with no bound on a process that is not a loop.
The other direction is unbounded too: two waiters that both find no live loop both spawn one.
The second loop's first child gets refused as busy, so that loop stops with `busy` and its waiter reports a busy stop while a loop is running.
The focus asks whether the pid-file liveness bound is concrete, and it is not.

**Fix:** In D2, state how liveness is proved beyond a bare pid.
For example, the pid file records the process start time or a token the loop holds, checked the way `internal/proc`'s liveness helpers allow.
Also state that the spawn is exclusive, made under a lock or by an exclusive create of the pid file.
Add both cases to the acceptance row on a second `--until-stop`.

### [MEDIUM] Unwatched-envelope delivery has no staleness bound

**Location:** `_lyx/discussion/decision-record.md`, D2; Acceptance criteria, the second `--until-stop` row.

**Issue:** "An envelope of a loop that ended unwatched is delivered by the next invocation and then retired" has no limit on how old that envelope may be.
Between the loop's end and the next `--until-stop`, the run can move on: `lyx loom resume`, a `goto`, a Go driver step, or an operator step.
The next invocation then prints a stop the run is no longer at, and the driver acts on it: a repair, a fork, or a report.
The focus names this delivery path as a bound to check.

**Fix:** In D2, state what makes an envelope current.
For example, its `current_producer` and `history_length` must still match the status file.
State that a stale envelope is retired without being delivered and a fresh loop is spawned.
Add an acceptance row for a stale envelope.

### [MEDIUM] The closed `loop` key set disagrees with the Decisions that fill it

**Location:** `_lyx/discussion/decision-record.md`, D1, D4, D7, D8; Acceptance criteria.

**Issue:** D1 closes the `loop` key set: `steps`, the first producer (key unnamed), the stop cause (key unnamed), `detail`, `trace_copy`, `stderr_path`, `current_producer`, `history_length` and `interrupt_policy`.
Other Decisions put keys on the envelope that this set lacks or places elsewhere:

- D4 carries "whether the status file moved", which has no key.
- D7 puts `trace_copy`, `stderr_path`, `envelope_path` and the friction and scratch directories *inside* `loop.detail`, while D1 makes the first two top-level `loop` keys.
  `envelope_path` appears nowhere in D1.
- D8 and the acceptance row record the transient re-step "on the loop envelope", with both failures, but it has no key.
- The acceptance criteria name the cause key `loop.stop`, which D1 never names.

Two more gaps sit in D1 itself.
A bare `pause` also ends as `paused`, and D1 does not say whether that is `halted` or `stop-condition`.
For an `interrupted` stop there is no child envelope, so "the stop step's short envelope (success or error form, unchanged keys)" does not say which error-form keys the loop fills (`trace_file`, `transient`, `friction`).
The acceptance criterion says "the closed set above", so a plan writer cannot meet it as written.
On the focus's count question, D4's "five to six" is correct and not a finding.
`KindBootstrap` is already one of the five declared kinds in `internal/shedverbs/step.go`'s `StepKinds`, so `interrupted` is the sixth.

**Fix:** Restate the closed `loop` key set once in D1, with every key named.
That includes the cause key, the first-producer key, the status-moved flag and the re-step record.
Say whether the paths sit in `detail` or at the top level, with D7 pointing at D1 instead of listing its own.
Name the cause for a bare pause.
State which error-form keys an `interrupted` stop carries and what fills them.

### [MEDIUM] Teardown kills the loop after the reed session ends

**Location:** `_lyx/discussion/decision-record.md`, D11.

**Issue:** D11 kills the live loop "after ending the reed session and before any worktree is removed".
The loop does not depend on the reed session: D2 makes it detached, and its children are in the loop's own session.
In the window between ending the reed session and the kill, which includes `pairteardown`'s quiet wait for the driver, the loop keeps stepping.
A child step can start a producer that spawns a new strand into the pair being torn down.
`PATTERN-pair-teardown` only requires the reed session to end before worktree removal.
Killing the loop first is compatible with that rule and closes the window.

**Fix:** In D11, kill the loop before ending the reed session, or at least before the quiet wait, and keep "before any worktree is removed".
The `pairteardown` acceptance row then checks that no step runs after teardown begins.

### [MEDIUM] The new invariant's domain is wider than what implements it

**Location:** `_lyx/discussion/decision-record.md`, D5; Scope, Out.

**Issue:** D5 words `PATTERN-failure-stop-visible` as "a driven run that stops on a failure leaves its status file non-running before the driver wakes".
Only the `--until-stop` loop implements it.
Scope Out excludes "any change to the Go driver `lyx loom run` beyond the engine honouring the stop condition".
Today only the engine writes `StateFailed` (`internal/shedengine/run.go`), so a pre-step refusal on another path, such as the Go driver or a bare `lyx loom step`, still leaves `running`.
As worded, the invariant is false from the commit that adds it, and review would enforce it against code this task does not change.

**Fix:** Scope the invariant to what this task makes true, such as "a run driven by `--until-stop`", or show that the other driving paths already comply.
Alternatively, move the arming-refusal write into the shared pre-step path so that every driver gets it, and bring that into Scope.

### [LOW] The lifetime of a condition that never fires is unstated

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** A condition is cleared only when it fires or by `--clear`.
A `--before X` set after X has run, or one a `goto` routes past, stays recorded and fires on a later rework pass, possibly much later.
`internal/shedengine/goto.go` already clears `PauseRequested` on a goto, but D3 does not say whether a goto clears the conditions too.
The halt case is decided: a halt leaves `after` recorded.
The goto and never-reached cases are not.

**Fix:** In D3, decide whether a goto clears the conditions, as it clears the bare request, and state that a condition otherwise lasts until it fires or is cleared.
Add the goto case to the stop-condition acceptance row.

### [LOW] D8's second-failure rule is ambiguous

**Location:** `_lyx/discussion/decision-record.md`, D8.

**Issue:** "A second failure at the same producer stops" can mean two consecutive failures, which is the re-step failing, or any two transient failures at that producer during the loop's life, including one on a later rework pass after a success.
The two readings bound the loop differently, and the acceptance row does not settle which applies.

**Fix:** State that the count is consecutive: the re-step itself failing stops the loop, and a success resets it.
Alternatively, state the other reading if that is intended.

### [LOW] The PATTERN cap entry is a Constraint, pins a number, and carries run history

**Location:** `_lyx/discussion/decision-record.md`, `### Added after Discussion (parent, 2026-10-10)`.

**Issue:** This entry binds every card's edits to `PATTERN.md`, not a design choice of the loop.
It sits unnumbered under `## Decisions` while Scope does not mention it, which is the focus's Scope-against-Decisions check.
"At 16375 of its 16384-byte cap" pins a count that changes with every PATTERN edit; the file is 16375 bytes today, so it is correct now and goes stale on the next edit.
The completeness-before-leanness test passes: the actionable requirement is the 16200-byte ceiling, offsetting by shortening, detail moved to the background file, and no invariant dropped.
The rationale's run history, which names three runs that hit `TestOverview_RealFilePassesCheck` at Publish, is deliberation narrative.

**Fix:** Move the requirement into `## Constraints` as one line.
Keep the 16200 ceiling and the 16384 cap, and drop the current byte count.
Move the run history to `support-log.md`.

### [NIT] D10 restates D4's stderr routing

**Location:** `_lyx/discussion/decision-record.md`, D10, third line.

**Issue:** "The SIGQUIT dump and every child's stderr go to the step's captured stderr file" repeats D4's "its goroutine dump lands in the captured stderr file, never on the driver's screen".
D2's log and D7's `stderr_path` cover the rest.
The same fact stated twice can drift.

**Fix:** Drop the line from D10, or keep it there and have D4 point to D10.

## Verdicts on the focus's other named instances

- D4 watchdog kill: bounded by the idle window, its one-minute floor and the activity reading. No finding.
- D9 fork: bounded by one fork per stop, edits limited to the report and records, a single returned action run under the repair cap, and other subagent types still denied. No finding.
- D8's re-step is bounded to one, apart from the ambiguity in F9.
- Rationale lines inside Decisions: the writer stencil allows "Decision + Rationale" in the record, so they are not relocation findings.
  Every requirement in `support-log.md`'s Interview, such as the shared write-to-clear flag and the arming refusal's cause, is also stated in a Decision.
- Scope against Decisions: every Scope "In" bullet is settled by a Decision.
  D10's SIGQUIT routing is covered by the watchdog bullet, apart from the duplicate in F11.
  The cap entry is F10.
- Auto-mode assumptions:
  - The shell-lifetime assumption (1) and the batten-reading assumption (2) only change cost, and the record covers both: (1) through its own text, (2) through a stated fallback.
  - The message-delivery assumption (3) is one today's driver already relies on, so this task adds no new exposure.
  - The fork assumption (4): the claude engine already supports fork-authorized runs (`internal/shuttleengine/claudeengine/audit.go`, `background.go`), and Open risks names a fallback if a fork costs more than reading in place.
  - The orch assumption (5) changes nothing if false.
  None of them makes a Decision wrong without a stated consequence.
