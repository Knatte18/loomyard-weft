# Decision record — driver-context

## Goal

The loom driver steps in only when something goes wrong, and its context stays under 100k tokens for a whole run.
Go runs the shed steps in a loop and wakes the driver only at a stop: a halt (`blocked`, `awaiting`, `paused`, `done`), a step error, a hung step, or a stop condition the orchestrator asked for.
A failed step never parks silently, and a woken driver at an awaiting PR-Gate steps the run instead of judging the pull request on GitHub (#488, #491, and the step watchdog from `verify-watchdog`, #420).

## Scope

In:

- A loop mode of the generic `step` verb, `lyx shed step <run-id> --until-stop` (and so `lyx loom step --until-stop`), that runs one step after another until a stop and answers with one envelope.
- The loop survives the driver's shell: it runs detached, and the command the driver runs is a waiter on it.
- A stop condition recorded in the run's status file by the generic `pause` verb: `--before <producer>`, `--after <producer>`, `--clear`.
- A step watchdog in the loop: a step that shows no activity for a configured window is killed and reported as a stop.
- A rich stop answer: the stop's envelope carries what went wrong, the trace lines that matter and the paths to read further, and the stop step's trace is copied out of retention's reach.
- A pre-step refusal (arming or config failure before the producer runs) carries a `kind`, leaves the status file non-running, has its refusal row, and so reaches the parent.
- One transient re-step done by the loop instead of the driver.
- The driver stencil rewritten around the loop: no per-step trace reads, no baseline status read, no artifact reads, a one-shot fork for the heavy work at a stop, a repair guide read on demand, the stop-condition command, and the PR-Gate rule.
- A slimmer driver start: `scribe:prose` dropped from the driver's skills.
- Compact envelopes: the step envelope's `progress` drops `remaining` and its empty keys.
- Trace noise: the plan gate's informational findings logged as a count plus a file, the stencil drift notice logged once per process as one line.
- The status strand kept on llm-driven runs.
- `pairteardown` counts a live loop as busy and stops it before ending the pair's session.
- The orch stencil names how to ask a driver for a stop at a point.

Out:

- Lower effort or thinking for Sonnet drivers (the board's idea 7, optional).
- The hub-wide interrupting pause (`lyx-halt`), message routing (`run-message-routing`), and any change to the Go driver `lyx loom run` beyond the engine honouring the stop condition.
- Keeping Claude Code's skill listing or tool schemas out of the driver session: the provider injects them, lyx has no lever there.
- Changing batten's own loop or its notices.

## Decisions

### D1. The loop is a mode of the generic `step` verb

`lyx shed step <run-id> --until-stop` loops one step after another and prints one envelope at the stop.
Each iteration runs the single-step verb as a child process of the same binary and subtree, with the loop's own argv minus the flag, from the same cwd, so every per-step hook, record, lock window and trace stays exactly what a single `step` does today.
The loop mints a fresh trace id for each child and hands it through `LYX_TRACE_ID`, so each child keeps its own trace group instead of inheriting the loop's, and the loop knows the child's trace file without an envelope.
The loop stops at: a step whose `continue` is false; a step error envelope; a child that ended without an envelope (interrupted); a stop condition that fired (seen as `paused`).
The loop's final envelope is the stop step's short envelope (success or error form, unchanged keys) plus one `loop` object whose key set is closed at:

- `steps`: the number of child steps run.
- `first_producer`: the producer the loop's first child stepped.
- `stop`: the stop cause, one of `halted` (`continue` false on `blocked`, `awaiting`, `done`, or `paused` from a bare pause request), `stop-condition` (`paused` from a `--before` or `--after` condition, D3), `error`, `interrupted`, `busy`.
- `detail`: the stop's text, filled per D4 and D7.
- `status_moved`: whether the status file's `current_producer` or `history_length` changed during the stop step.
- `restep`: the transient re-step of D8 (the producer and the first failure's error and transient class), absent when none ran.
- `current_producer`, `history_length`, `state`: read from the status file at the stop.
- `interrupt_policy`: the stop row's interrupt policy.
- `trace_copy`, `stderr_path`: the copied trace and the captured stderr of the stop step (D7).

An `interrupted` stop has no child envelope, so the loop fills the short error form's closed key set itself: the error is `loop.detail`'s cause, `kind` is `interrupted`, `transient` is empty, `run_id` is the run's, `trace_file` is the child's trace file, and `friction` is the halt note's status (D5) when one was written.
The loop also writes the full error envelope of that stop as a step record in the run's steps directory, carrying the full form's location keys, and `envelope_path` names that record.
Rationale: the driver's contract stays the step envelope it already branches on; the child keeps the single-step verb untouched and gives per-step traces, per-step records and a killable process for free; the generic verb body belongs in `internal/shedverbs` (PATTERN-shed-verb-set), and the flag lands on every subtree that arms it.

### D2. The loop runs detached; the driver's command is a waiter

The driver's shell can be killed with its process group after a bounded time (the board's `orch-owns-the-loop` note, 2026-10-07), while a run lasts hours.
So `--until-stop` first spawns the loop detached, with stdout and stderr in a log under the run's steps directory, then waits for the loop's envelope file in that directory, prints it and retires it.
The spawn uses `internal/proc`'s `DetachBreakaway`: on Linux a new session, which the shell's process-group kill does not reach; on Windows a new process group broken away from any job around the driver's shell, so closing that job does not take the loop.
Where that job forbids breakaway the spawn is refused, and the waiter spawns with `Detach` instead and logs it; that loop lives only as long as the shell's job (Open risks).
The loop holds a loop lock in the steps directory for its whole life, through `internal/lock`, and that lock is the liveness record: the OS releases it when the loop dies, so a stale record or a reused pid never reads as a live loop.
A second `--until-stop` that finds the loop lock held spawns nothing and waits on the live loop.
The spawn is exclusive: a loop that cannot take the loop lock at start exits at once, before any step and without an envelope, and its waiter waits on the loop that holds the lock.
A loop is gone only when the loop lock is free and no envelope exists; a waiter whose spawned loop exited while the lock is held waits on the holder.
The loop's pid file, beside the lock, records its pid and process start time and serves only to kill it on Linux (D11); a kill acts only while the loop lock is held and the start time still matches.
On Windows, where `proc.StartTime` reads nothing, the loop's identity is a kill-on-close Job Object the loop creates at start under a name derived from the run's steps directory path, unique across hubs, and assigns itself to; the per-child jobs of this Decision nest inside it.
The name exists only while a handle to the job is open, and the loop holds the only lasting one, so opening the job by name reaches the live loop and its whole tree, never a reused pid.
A waiter killed before the envelope arrives loses nothing: the next waiter prints the envelope.
The loop writes its pid file only after it takes the loop lock, and removes it only after its envelope is on disk.
So a pid file with the loop lock free and no envelope marks a loop that died without one: a crash, an outside kill, an OOM kill or a reboot.
A waiter whose loop is gone and an invocation that finds that state both stop the run as `interrupted` with `loop.detail` cause `loop-exited`, the error form of D1 and the `interrupt_policy` of the status file's `current_producer` row.
That stop first kills what is left of the dead loop's step tree (D11's tree kill), then writes `failed` through D5's function under D5's bound, writes its envelope as the loop's envelope file, and retires the pid file.
When D5's write meets another holder of the run lock, the stop ends `busy` instead: it keeps the pid file, writes no loop envelope and prints the `busy` stop, so the next invocation retries the dead-loop stop.
Bound: an invocation never spawns a loop over a dead loop's pid file, so a half-run producer is re-invoked only by the driver's interrupted rule, under its repair cap.
An envelope of a loop that ended unwatched is delivered by the next invocation only while it is current, meaning its `loop.current_producer`, `loop.history_length` and `loop.state` still match the status file; either way it is then retired.
A stale envelope is retired undelivered and a fresh loop is spawned: a resume, which writes `running` over the same producer and history, changes `state`, and a `goto` or another driver's step changes the producer or the history.
The wait wakes on a file event through `internal/fswatch` where available, else polls at a fixed interval of seconds, never below one second (PATTERN-daemon-wakeups: the waiter and the loop's watchdog join the declared list).
A child step is tied to the loop's life, never to the driver's shell, by the mechanism `internal/gatecli` gives `lyx gate test`'s child: on Linux a parent-death SIGKILL (`Pdeathsig`), spawned from a goroutine locked to its OS thread for the child's life, since the signal follows the spawning thread; on Windows a kill-on-close Job Object the loop creates per child, which the OS closes when the loop dies.
That mechanism moves into `internal/proc`, and `internal/gatecli` uses it from there.
Each child runs in its own process group, whose id is the child's pid.
On Linux the parent-death signal reaches the direct child only, so the child's descendants are reached by the tree kill: SIGKILL to the recorded process group, acting only while a live process reports that group in `/proc/<pid>/stat` with a start time at or after the recorded child start.
Linux keeps a group id unassigned while any member lives, so a matching member belongs to the step's tree, and the gate holds after the group leader is dead.
On Windows the child's Job Object holds its descendants, and the loop's job holds that one, so the tree dies with the loop.
A descendant that leaves the group or job on its own is reached only by its own parent-death tie, as `lyx gate test`'s child is.
Session membership kills nothing: a detached session has no controlling terminal to hang up.
The loop records the in-flight child's pid and start time in its pid file, beside its own, for teardown (D11), with the status file's `current_producer` and `history_length` at the child's start, so a `loop-exited` stop fills `loop.status_moved`.
The loop logs each child spawn and each kill through `internal/logger` (PATTERN-spawn-observability).
Bound: the loop is one more long-lived `lyx` process per run, like the Go driver and the daemons; a loop stops on its own when the status file vanishes, when `pause` fires, or when `pairteardown` kills it (D11).

### D3. A stop condition lives in the status file and is honoured by the engine

`lyx shed pause <run-id> --before <producer>` and `--after <producer>` record the condition on the status file, through the same status lock the bare `pause` uses; `--clear` removes both conditions and leaves `pause_requested` alone.
Bound: `--clear` lets through only what a condition would have stopped; an operator's bare pause request stays recorded, and Shed remains its only writer of `false`.
A bare `pause` keeps its meaning: stop at the next boundary.
The two condition fields form their own ownership class, outside-set and both-cleared: an outside actor sets them and removes them through `pause`, and Shed writes them cleared only in the persist that records `paused` for the condition and in a `goto`'s persist, which also clears `pause_requested` today.
Each condition is one slot: `--before Y` over a recorded `--before X` replaces it, and `--clear` removes both conditions whoever set them.
Bound: an actor can retarget or drop a condition another actor set, never the bare pause request, and the verb's answer names the target it replaced or removed, so the actor sees what it overrode.
`internal/shedengine` consumes the conditions where it consumes `pause_requested`: at the boundary check, `before` fires when `current_producer` equals its target; after the producer returns with a routed running outcome (Done to `OnDone`, Stuck to `OnStuck`), `after` fires when the producer that ran equals its target, and the one persist writes `paused` with `current_producer` at the routed row.
A condition fires once and is cleared in the persist that records `paused`; a halt outcome (`blocked`, `awaiting`, `done`) wins over `after` and leaves the condition recorded.
Otherwise a condition lasts until it fires, `--clear` removes it or a `goto` clears it, so one whose target already ran fires on the next pass that reaches the target.
The paused `StepResult` carries a `Reason` naming the condition ("paused before Webster, as requested"), so the envelope tells the driver which stop this is.
The target must name a producer of the recipe's routing projection the verb is told; another name is refused listing the producers.
Setting a condition never touches the loop or a running producer: the producer in progress finishes, and the engine reads the file at the next boundary.
Rationale: the engine already owns the shared write-to-clear pause flag and both the Go driver and the loop go through it, so one site honours the condition for every driver; the status file is the run's own state the board asks for.

### D4. The loop watches each step

The loop kills a child step that shows no activity for `step_idle_timeout_min`, a new `loom.yaml` key with a default of 60 minutes and a floor of one minute, told to the loop through its spec.
Activity is told as a hook the arming module fills: the newest of the step's trace group's mtime and the run's agent activity, with a live verify wait marker counting as activity, the same reading batten's quiet notice uses.
The kill takes the child with its descendants, so a hung grandchild dies too: on Linux a SIGKILL to the child's process group, on Windows a termination of the child's Job Object (D2); on Linux the child gets SIGQUIT first so its goroutine dump lands in the captured stderr file, never on the driver's screen.
A child that ended without an envelope, killed or crashed, is the sixth step kind, `interrupted`, with `loop.detail` naming the cause (`watchdog` with the idle time, or `exited` with the exit status).
The loop never re-invokes an interrupted step itself: the envelope carries `loop.interrupt_policy` and `loop.status_moved`, and the driver's existing interrupted-invocation rule decides, under its repair cap.
Rationale: a per-step deadline would kill legitimate hours-long rows (Webster); idleness over the same reading batten already trusts is the bound that distinguishes hung from slow.
This changes PATTERN-shed-verb-set: the step kind vocabulary grows from five to six, and `internal/shedverbs`' import allowlist gains `internal/proc` and `internal/fswatch`.

### D5. A failure stop leaves the status file non-running

When the loop stops on an error envelope or an interrupted step while the status file still reads `running` (a pre-step refusal, a child that died before its persist, a loop that died without an envelope, D2), the loop calls one `internal/shedengine` function that writes `failed` with the error text and the way forward, history untouched, and the engine's resume write clears it on the next step.
Bound: the function writes only while it holds the run lock that the busy refusal reads, taken without waiting, and only while the file under that lock still reads `running`; when another holder has the lock, such as a Go driver, an operator's `step` or another loop that took the run after the child ended, it writes nothing and the loop stops as `busy`.
So `State` and `Error` stay written by Shed alone, and the loop never overwrites a live step.
A busy refusal writes nothing: another holder owns the run.
An interrupted stop under loom also writes the halt friction note a `failed` halt writes (PATTERN-friction-capture), through the arming module's hook.
The driver's stop report and park follow as for any halt, and batten's state-changed notice reaches the hub.
This is a new cross-cutting invariant, `PATTERN-failure-stop-visible`: a run driven by `--until-stop` that stops on a failure, with no other holder of its run lock, leaves its status file non-running before the waiter returns, so `lyx loom resume` never answers "already running" for such a stopped run and no failure waits for the quiet notice.
The Go driver `lyx loom run` and a bare `step` stay outside the invariant (Scope, Out).
Rationale: the `bolt-and-store` silent park came from a refusal that touched no state.

### D6. A pre-step refusal carries a kind and a trace

A refusal raised while arming the step (seed read, recipe lookup, config load, the pre-step hook) is reported with `kind: bootstrap`, with the trace armed so the envelope's `trace_file` holds the refusal, instead of a bare error before the verb runs.
The missing-hub-config case gets its row in the shed section of the refusal spec, with the way forward `lyx config reconcile --apply` in the hub, then the step again (PATTERN-refusal-way-forward).
Bound: the driver's disposition for `bootstrap` is the repair path, whose repairs are a closed list of lyx verbs; a config fix is outside it, so the driver escalates with the way forward, never edits config.

### D7. The stop answer is rich, the success answer is short

An error or halt stop carries `loop.detail`: the step's error or reason, the transient class, and the WARN and ERROR lines of the stop step's trace plus its last lines, capped at a fixed line and byte budget.
The paths to read further are `loop.trace_copy` and `loop.stderr_path` (D1) and the short envelope's `envelope_path`, whose full record names the friction and scratch directories.
The loop copies the stop step's trace file into the run's steps directory as `trace_copy` once the child has exited; a copy outside the trace directory is never swept, so retention needs no pin.
A successful iteration prints nothing; the final `done` envelope is the short success envelope.
The step envelope's `progress` drops `remaining` and every empty key; the status envelope keeps `remaining`, since the driver names producers from it.
Rationale: the driver reads one envelope per stop and opens a trace only to dig deeper; the trace copy lives beside the step record under `.lyx` (PATTERN-durable-vs-ephemeral-state).

### D8. The loop makes the one transient re-step

An error envelope with a non-empty `transient` gets one immediate re-step by the loop, recorded as `loop.restep`; if the re-step itself fails, the loop stops with the second failure as the stop step's envelope.
The count is consecutive: a successful step resets it, so a later transient failure at the same producer gets its own re-step.
Bound: at most one loop re-step per failure, never two in a row at one producer.
The transient marks and their sites are unchanged (PATTERN-transient-stop); the policy moves verbatim from the stencil to Go.
The binary-change re-step stays with the driver, since it is a parked driver's concern.

### D9. The driver stencil is rewritten around the loop

`contracts/stencils/shed/shed-template-driver.md` changes to:

- The loop command is the one background job; the driver reads the envelope at its exit and never reads a trace, an artifact or a baseline status first.
  A job that ends without an envelope is re-run: the re-run waits on a live loop, or reports a dead one as an interrupted stop (D2).
- `done` tears down; `blocked`, `awaiting`, `paused` and `busy` keep their hand-backs.
- At an error or interrupted stop the driver spawns one fork of itself with the envelope: the fork reads the trace copy, stderr, child traces and the repair guide, writes the stop report and any repair records, and returns a short conclusion and one action (a named repair verb, a re-launch, or an escalation); the driver runs the action.
  Bound: one fork per stop; the fork edits nothing but the report and records.
- The repair catalogue, the stranded-branch exception, the binary watch and the interrupted-invocation rule move to a second stencil, `shed-template-driver-guide`, whose path the prompt tells through a marker; the driver stencil points at it (PATTERN-producer-pointer-rule) and the fork reads it.
- A message asking for a stop at a point becomes `lyx shed pause <run-id> --before|--after <producer>`, with producer names from the envelope's `progress` or `lyx shed status`; the driver answers the sender that the stop is recorded, and the stop itself reaches the parent by the existing notify rules.
- At a resume line over an `awaiting` run the driver launches the loop at once; it never inspects the pull request or judges a decision, since the gate reads the recorded decision (#491).
- The repair cap's key comes from the loop envelope's `current_producer` and `history_length`.

`driverSkills` drops `scribe:prose`, and `driverSpec` sets `ForkSubagents` (fork subagents allowed, other subagent types still denied by the claude engine).
`contracts/stencils/orch/orch-template-role.md` gets one line: to stop a run at a point, message its driver with the producer name.

### D10. Trace noise

The plan gate logs informational findings as a count and writes them to a file under the run's scratch directory the WARN names.
The stencil drift notice is one Warn per process listing the drifted stencils.

### D11. Teardown and the status strand

`internal/pairteardown`'s quiet rule counts a pair whose loop lock is held as busy, so `fabric remove`'s bounded wait refuses a pair with a live loop, as it refuses a busy driver.
The session-end step, as its first act, kills a live loop of the pair and its step tree.
On Linux it kills the loop by its pid file while the loop lock is held and the start time matches (D2), then the in-flight child's process group the pid file records under D2's tree-kill gate, which reaches the child's descendants though the child already died with the loop.
On Windows it opens the loop's job by name and terminates it, which ends the loop, the in-flight child and its descendants at once.
It then waits a bounded time for the run lock to be released, and fails with the holder's pid when it is not, so the reed session ends only after the in-flight step is dead and before any worktree is removed (PATTERN-pair-teardown order kept); batten's zero-wait path reaches it too.
Bound: no step starts in a pair once its session-end step begins, and no step started before it outlives it.
`lyx loom start` keeps the status strand on llm-driven runs instead of removing it, since the driver pane no longer shows the run's steps.

### Added after Discussion (parent, 2026-10-10): Watchdog counts every seat strand of a multi-seat row

Decision: The step-idle watchdog's activity source counts activity from every seat strand of a MultiLLMProducer row (the chair and each advisor), not only from a single producer strand.

Rationale: Run discussion-multi-llm moves Discussion-Write onto MultiLLMProducer with a chair and advisors; a watchdog reading one producer strand would see a working Discussion as idle while advisors answer.

### Added after Discussion (parent, 2026-10-10): Loop process lifetime: settle round-3 findings at Plan

Decision: Plan settles round 3's F1-F6 as stated: (F1) teardown and a dead loop kill the step's whole recorded process group, gated on the loop's start time and the held loop lock rather than the group leader's, and the teardown and loop-killed acceptance rows assert that a grandchild of the step is dead; (F2) on Windows the kill uses a creation time read through GetProcessTimes added to internal/proc, or a named job object the teardown terminates, with its bound against pid reuse stated; (F3) the loop is spawned with proc.DetachBreakaway on Windows; (F4) a waiter retires the envelope it prints, and the currency check includes the status file's state, so an envelope recorded at a halt is stale once the file reads running; (F5) a dead-loop stop that ends busy keeps the pid file and writes no envelope, so the next invocation retries it; (F6) the loop is gone only when the loop lock is free and no envelope exists.

Rationale: Three Discussion rounds each found new process-lifetime detail in the same area while the design's goal and scope held; the fixes are stated precisely enough to plan and test, so a fourth round would add cost without changing the design.

### Added after Discussion (parent, 2026-10-10): Teardown mark survives dead-loop stops

Decision: Card 12's deadLoopStop keeps a pid file whose record names the teardown as its loop id instead of retiring it, and writes no loop envelope over it, so every invocation until RemovePair stops loop-exited and spawns nothing; card 18 states the mark survives every dead-loop stop; TestUntilStop_DeadLoopStopsLoopExited gains rows for the waiter on the teardown-held lock and for a second and third invocation over the mark.

Rationale: Plan round 4 MEDIUM F1, the review's own first fix; accepted at budget rather than a fifth round.

## Constraints

- PATTERN overview cap (added by the parent, 2026-10-10): every card that adds or edits a `PATTERN.md` overview line keeps the overview at most 16200 bytes on the task branch, under its 16384-byte cap; added text is offset by shortening wording in the same or another overview line, with detail moved into the `pattern/` background file, and no invariant is dropped.
- PATTERN-shed-verb-set: the loop body, the flag and the pause flags live in `internal/shedverbs`; the kind vocabulary becomes six and the allowlist grows, both stated in the entry and its background file in the same commit.
- PATTERN-daemon-wakeups: the waiter and the watchdog are listed unbounded waits; one-second floor, backoff, `fswatch` as the event source.
- PATTERN-refusal-way-forward: new rows for the pause target refusal, the arming refusal with kind, the missing hub config, and the interrupted stop; each reached by a test.
- PATTERN-driver-choice-single-site: nothing new reads the seed's driver.
- PATTERN-transient-stop, PATTERN-friction-capture, PATTERN-stencil-ownership, PATTERN-role-skills-typed, PATTERN-parent-directive, PATTERN-edit-directive: unchanged and obeyed; the new guide stencil is read through `stencilstore` from the told directory.
- PATTERN-spawn-observability, PATTERN-pair-teardown, PATTERN-durable-vs-ephemeral-state, PATTERN-lyxdirs-single-declarer, PATTERN-shed-run-directory: the loop's files (loop lock, pid, log, envelope, trace copy, stderr) are declared by `internal/shedrun` under the run's ephemeral steps directory.
- PATTERN-test-speed and PATTERN-test-economy: the loop is tested at tier 1 through a seam for the child runner, the clock and the activity hook; one `integration` test runs a real child; `cmd/lyx/testdata/test-budget.yaml` rows rise where needed.
- `loom.yaml` gains `step_idle_timeout_min`; the hub reconcile seeds it on the binary change (PATTERN-binary-change-reconcile).
- `internal/shedengine`'s `Status` field-ownership contract names the two condition fields as their own outside-set, both-cleared class (D3) and the failure-stop write as a Shed write under the run lock (D5).
- Docs in the same commit: `internal/shedverbs/doc.go`, `internal/shedengine/doc.go`, `docs/overview.md` where the execution stack mentions the driver, `PATTERN.md` and `pattern/` for the changed and new entries, `contracts/specs/refusal-spec.md`, and the claims test for the stencils.

## Auto-mode assumptions

- The driver's background shell can be killed after a bounded time with its process group, as the board records; the detached loop is designed for that and costs little if the limit is generous.
- Batten's agent-activity reading is reachable from `internal/loomcli` for the loop's activity hook without importing `battenshed`; if not, the hook reads the same sources directly.
- Claude Code delivers a message to the driver session while its background job runs, and the job's exit wakes it, as the stencil already relies on.
- The claude engine's fork mode gives the driver a fork that inherits its context and returns a text result; a fork started from a context under 50k stays under 100k.
- The orchestrator keeps asking for stops in plain words to the driver; it may also run the pause verb in the pair itself.

## Open risks

- The watchdog window is a guess: 60 minutes idle may still kill a legitimately silent step (a long git transfer with no trace writes, a verify whose wait marker is stale); the first runs tune the key.
- Killing a step whose producer holds external state (a Webster batch mid-commit, a pushed branch) leaves what the single-step kill leaves today; the interrupted rule and the repair guide cover it, nothing new is lost.
- A detached loop outliving a dead driver session steps the run unattended until its next stop; batten's driver-dead notice covers the watched case, and the status strand shows it.
- The parent-death signal follows the spawning OS thread, so a loop that spawns a child from an unlocked goroutine loses the tie on Linux when the runtime retires that thread; the loop's own tests pin a child's death on the loop's kill (round 2).
- A loop that dies between a child's persist and its own envelope is reported as `loop-exited` although the step finished; `loop.status_moved` shows it, and the driver's interrupted rule reads it before any re-invoke (round 2).
- On Windows, a driver shell inside a job that forbids breakaway gets a loop spawned with `Detach`, which dies when that job closes; the waiter logs the fallback, and the next invocation reports the dead loop as `loop-exited` (round 3).
- `history_length` is not a strict move counter, since the engine folds a repeated budget-exempt Stuck at one producer into the last entry; an unwatched envelope whose producer, history length and state all recur after a resume is delivered although stale, and the driver's next launch steps on (round 3).
- An actor can retarget or clear another actor's stop condition; the pause verb's answer names what it overrode, and nothing tells the actor that set it (round 2).
- The fork's context cost depends on the provider's cache behaviour; if a fork turns out costlier than reading the trace in place, the stencil falls back to reading in place for small traces.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - dead-loop-stop-busy-pid-file-fate (class decision, severity MEDIUM, open in rounds 3)
  - envelope-currency-blind-to-resume (class design, severity MEDIUM, open in rounds 3)
  - teardown-kills-loop-after-reed-session (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - windows-detach-stays-in-job-object (class design, severity MEDIUM, open in rounds 3)
  - windows-start-time-gate-never-matches (class design, severity BLOCKING, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-4-review.md; fixer report: _lyx/reviews/plan/round-4-fixer-report.md.
  - session-end-respawn-window (class design, severity MEDIUM, open in rounds 2, 3, 4)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `--until-stop` runs steps until `continue` is false and prints one envelope whose step keys equal the stop step's short envelope and whose `loop` key set is D1's closed set; a bare pause stops with `loop.stop: halted` and a fired condition with `loop.stop: stop-condition`.
- Each child step writes its own trace group, distinct from the loop's and from every other child's.
- A stop condition `--before X` set during a run pauses the run with `current_producer` X before X runs; `--after X` pauses once X returned, at the routed row, for a Done and for a bounced Stuck; a halt of X leaves `--after X` recorded; `--clear` removes both and leaves a recorded bare request in place; a second `--before` replaces the first and the answer names the replaced target, as `--clear`'s names the removed ones; a `goto` clears both; an unknown target is refused listing the producers; the Go driver `lyx loom run` honours the same conditions.
- The pause reason names the condition and reaches the envelope.
- A child step that shows no activity for the window is killed with its process group, its stderr captured, and the loop stops with `kind: interrupted`, `loop.stop: interrupted`, the policy of the row and the status file written `failed`; a child that exits without an envelope stops the same way with cause `exited`.
- A pre-step refusal (missing hub config, unreadable seed) answers with `kind: bootstrap`, a trace file holding the refusal, and the status file written `failed` when one exists; the missing-config row's way forward is in the message.
- A busy refusal stops the loop and writes no status.
- A failure stop while another holder holds the run lock leaves the status file untouched and stops with `loop.stop: busy`.
- A transient error is re-stepped once by the loop; a failing re-step stops with the first failure in `loop.restep` and the second as the stop step's envelope; a transient failure after an intervening success is re-stepped again.
- The stop step's trace is copied into the steps directory and named by the envelope; a sweep that removes the original leaves the copy.
- A second `--until-stop` while a loop is alive spawns no loop and prints the same envelope when the loop ends; a pid file left by a dead loop, its loop lock free, does not count as a live loop (it marks a dead one, see the row on a loop killed mid-step); of two loops spawned at once, only the one holding the loop lock steps, and the loser's waiter waits on the winner without writing `failed`; a waiter retires the envelope it prints, so the next invocation spawns a fresh loop; a current envelope of a loop that ended unwatched is delivered by the next invocation and then retired; a stale one, including a halt envelope after a resume wrote `running`, is retired undelivered and a fresh loop is spawned; the loop's children and their descendants die with the loop and survive the waiter's death; on Windows the loop survives the closing of a breakaway-permitting job around its spawner.
- The step envelope's `progress` carries no `remaining` and no empty keys, and the step envelope's other keys are unchanged; the status envelope keeps `remaining`.
- The plan gate's informational WARN carries a count and a path, and the file holds the findings; the drift notice appears once per process.
- `pairteardown` counts a held loop lock as busy, kills a live loop, its in-flight child and the child's descendants before ending the reed session, on Linux and on Windows, so the step tree is dead and no step starts after the session-end step begins, and tolerates an absent pid file.
- A loop killed mid-step takes its child and the child's descendants with it; the next invocation finds its pid file, spawns no loop, and stops with `loop.stop: interrupted`, cause `loop-exited`, the row's policy and the status file written `failed`; a waiter whose loop dies stops the same way; when another holder has the run lock, that stop ends `busy`, keeps the pid file and writes no loop envelope, and the next invocation retries it.
- An interrupted stop's `envelope_path` names a full record carrying the friction and scratch directories.
- `lyx loom start` on an llm-driven run ensures the status strand.
- The driver stencil and the guide stencil pass the claims test; the stencil names the loop command, the fork rule, the stop-condition command, the PR-Gate rule, and no per-step trace or artifact read; `driverSkills` is empty and the driver spec sets `ForkSubagents`.
- Every new refusal has its row and a test that reaches it; PATTERN and the background files state the six kinds, the allowlist, the listed waits and the new invariant.

## Notes for the plan writer

- The loop body sits beside `stepCmd` in `internal/shedverbs/step.go`; the child runner, the clock and the activity reading are seams on `Spec` or `Hooks` so the loop is tier 1 with fakes.
- `shedengine.stepLocked`'s step 3 and its routing arms are where the conditions are consumed; `Status` gains the two condition fields beside `PauseRequested`, and `contracts/specs/loom-status-spec.md` documents them.
- `internal/shedcli`'s pre-run reports arming errors through `output.Err` with no kind; the kind and the trace arming belong there or in a told pre-step hook.
- `internal/loomcli/start.go` step 4 removes status strands on the llm arm; `driverspec.go` holds `driverSkills` and the fork flag; `driverprompt.go` fills the markers, so the guide path is one more marker.
- `internal/battencli`'s `Activity` dependency for batten's notices is the reading to reuse for the watchdog hook.
- `internal/logger/retention.go` groups traces by id; the copy approach needs nothing there.
- `internal/loomshed/gates.go` logs the plan gate findings; `internal/stencilstore/reconcile.go` logs the drift notice per stencil.
