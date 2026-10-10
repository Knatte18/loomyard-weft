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
- Compact envelopes: `progress.remaining` dropped from the step envelope, empty keys dropped.
- Trace noise: the plan gate's informational findings logged as a count plus a file, the stencil drift notice logged once per process as one line.
- The status strand kept on llm-driven runs.
- `pairteardown` stops a live loop of the pair before removing worktrees.
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
The loop stops at: a step whose `continue` is false; a step error envelope; a child that ended without an envelope (interrupted); a stop condition that fired (seen as `paused`).
The loop's final envelope is the stop step's short envelope (success or error form, unchanged keys) plus one closed `loop` key: `steps` run, the first producer, the stop cause (`halted`, `stop-condition`, `error`, `interrupted`, `busy`), `detail`, `trace_copy`, `stderr_path`, `current_producer`, `history_length`, `interrupt_policy`.
Rationale: the driver's contract stays the step envelope it already branches on; the child keeps the single-step verb untouched and gives per-step traces, per-step records and a killable process for free; the generic verb body belongs in `internal/shedverbs` (PATTERN-shed-verb-set), and the flag lands on every subtree that arms it.

### D2. The loop runs detached; the driver's command is a waiter

The driver's shell can be killed with its process group after a bounded time (the board's `orch-owns-the-loop` note, 2026-10-07), while a run lasts hours.
So `--until-stop` first spawns the loop detached through `internal/proc`'s `Detach` (the precedent is `lyx loom start`'s Go arm), with stdout and stderr in a log under the run's steps directory, then waits for the loop's envelope file in that directory and prints it.
A second `--until-stop` while the loop is alive spawns nothing and waits on the live loop; the loop's pid file in the steps directory is the liveness record.
A waiter killed before the envelope arrives loses nothing: the next waiter prints the envelope, and an envelope of a loop that ended unwatched is delivered by the next invocation and then retired.
The wait wakes on a file event through `internal/fswatch` where available, else polls at a fixed interval of seconds, never below one second (PATTERN-daemon-wakeups: the waiter and the loop's watchdog join the declared list).
Each child step inherits the loop's session, so it dies with the loop and never with the driver's shell; the loop logs each child spawn and each kill through `internal/logger` (PATTERN-spawn-observability).
Bound: the loop is one more long-lived `lyx` process per run, like the Go driver and the daemons; a loop stops on its own when the status file vanishes, when `pause` fires, or when `pairteardown` kills it (D11).

### D3. A stop condition lives in the status file and is honoured by the engine

`lyx shed pause <run-id> --before <producer>` and `--after <producer>` record the condition on the status file, through the same status lock the bare `pause` uses; `--clear` removes both conditions and the bare pause request.
A bare `pause` keeps its meaning: stop at the next boundary.
`internal/shedengine` consumes the conditions where it consumes `pause_requested`: at the boundary check, `before` fires when `current_producer` equals its target; after the producer returns with a routed running outcome (Done to `OnDone`, Stuck to `OnStuck`), `after` fires when the producer that ran equals its target, and the one persist writes `paused` with `current_producer` at the routed row.
A condition fires once and is cleared in the persist that records `paused`; a halt outcome (`blocked`, `awaiting`, `done`) wins over `after` and leaves the condition recorded.
The paused `StepResult` carries a `Reason` naming the condition ("paused before Webster, as requested"), so the envelope tells the driver which stop this is.
The target must name a producer of the recipe's routing projection the verb is told; another name is refused listing the producers.
Setting a condition never touches the loop or a running producer: the producer in progress finishes, and the engine reads the file at the next boundary.
Rationale: the engine already owns the shared write-to-clear pause flag and both the Go driver and the loop go through it, so one site honours the condition for every driver; the status file is the run's own state the board asks for.

### D4. The loop watches each step

The loop kills a child step that shows no activity for `step_idle_timeout_min`, a new `loom.yaml` key with a default of 60 minutes and a floor of one minute, told to the loop through its spec.
Activity is told as a hook the arming module fills: the newest of the step's trace group's mtime and the run's agent activity, with a live verify wait marker counting as activity, the same reading batten's quiet notice uses.
The kill takes the child's process group through `internal/proc`'s group kill, so a hung grandchild dies too; on Linux the child gets SIGQUIT first so its goroutine dump lands in the captured stderr file, never on the driver's screen.
A child that ended without an envelope, killed or crashed, is the sixth step kind, `interrupted`, with `loop.detail` naming the cause (`watchdog` with the idle time, or `exited` with the exit status).
The loop never re-invokes an interrupted step itself: the envelope carries the row's `interrupt_policy` and whether the status file moved, and the driver's existing interrupted-invocation rule decides, under its repair cap.
Rationale: a per-step deadline would kill legitimate hours-long rows (Webster); idleness over the same reading batten already trusts is the bound that distinguishes hung from slow.
This changes PATTERN-shed-verb-set: the step kind vocabulary grows from five to six, and `internal/shedverbs`' import allowlist gains `internal/proc` and `internal/fswatch`.

### D5. A failure stop leaves the status file non-running

When the loop stops on an error envelope or an interrupted step while the status file still reads `running` (a pre-step refusal, a child that died before its persist), the loop writes `failed` with the error text and the way forward, through the status lock, history untouched, and the engine's resume write clears it on the next step.
A busy refusal writes nothing: another holder owns the run.
An interrupted stop under loom also writes the halt friction note a `failed` halt writes (PATTERN-friction-capture), through the arming module's hook.
The driver's stop report and park follow as for any halt, and batten's state-changed notice reaches the hub.
This is a new cross-cutting invariant, `PATTERN-failure-stop-visible`: a driven run that stops on a failure leaves its status file non-running before the driver wakes, so `lyx loom resume` never answers "already running" for a stopped run and no failure waits for the quiet notice.
Rationale: the `bolt-and-store` silent park came from a refusal that touched no state.

### D6. A pre-step refusal carries a kind and a trace

A refusal raised while arming the step (seed read, recipe lookup, config load, the pre-step hook) is reported with `kind: bootstrap`, with the trace armed so the envelope's `trace_file` holds the refusal, instead of a bare error before the verb runs.
The missing-hub-config case gets its row in the shed section of the refusal spec, with the way forward `lyx config reconcile --apply` in the hub, then the step again (PATTERN-refusal-way-forward).
Bound: the driver's disposition for `bootstrap` is the repair path, whose repairs are a closed list of lyx verbs; a config fix is outside it, so the driver escalates with the way forward, never edits config.

### D7. The stop answer is rich, the success answer is short

An error or halt stop carries `loop.detail`: the step's error or reason, the transient class, the WARN and ERROR lines of the stop step's trace plus its last lines, capped at a fixed line and byte budget, and the paths to read further (`trace_copy`, `stderr_path`, `envelope_path`, the friction and scratch directories from the full record).
The loop copies the stop step's trace file into the run's steps directory as `trace_copy` once the child has exited; a copy outside the trace directory is never swept, so retention needs no pin.
A successful iteration prints nothing; the final `done` envelope is the short success envelope.
The step envelope's `progress` drops `remaining` and every empty key; the status envelope keeps `remaining`, since the driver names producers from it.
Rationale: the driver reads one envelope per stop and opens a trace only to dig deeper; the trace copy lives beside the step record under `.lyx` (PATTERN-durable-vs-ephemeral-state).

### D8. The loop makes the one transient re-step

An error envelope with a non-empty `transient` gets one immediate re-step by the loop, recorded on the loop envelope; a second failure at the same producer stops.
The transient marks and their sites are unchanged (PATTERN-transient-stop); the policy moves verbatim from the stencil to Go.
The binary-change re-step stays with the driver, since it is a parked driver's concern.

### D9. The driver stencil is rewritten around the loop

`contracts/stencils/shed/shed-template-driver.md` changes to:

- The loop command is the one background job; the driver reads the envelope at its exit and never reads a trace, an artifact or a baseline status first.
  A job that ends without an envelope is re-run: the loop is alive and the re-run waits.
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
The SIGQUIT dump and every child's stderr go to the step's captured stderr file.

### D11. Teardown and the status strand

`internal/pairteardown` kills a live loop of the pair, by its pid file, after ending the reed session and before any worktree is removed (PATTERN-pair-teardown order kept).
`lyx loom start` keeps the status strand on llm-driven runs instead of removing it, since the driver pane no longer shows the run's steps.

## Constraints

- PATTERN-shed-verb-set: the loop body, the flag and the pause flags live in `internal/shedverbs`; the kind vocabulary becomes six and the allowlist grows, both stated in the entry and its background file in the same commit.
- PATTERN-daemon-wakeups: the waiter and the watchdog are listed unbounded waits; one-second floor, backoff, `fswatch` as the event source.
- PATTERN-refusal-way-forward: new rows for the pause target refusal, the arming refusal with kind, the missing hub config, and the interrupted stop; each reached by a test.
- PATTERN-driver-choice-single-site: nothing new reads the seed's driver.
- PATTERN-transient-stop, PATTERN-friction-capture, PATTERN-stencil-ownership, PATTERN-role-skills-typed, PATTERN-parent-directive, PATTERN-edit-directive: unchanged and obeyed; the new guide stencil is read through `stencilstore` from the told directory.
- PATTERN-spawn-observability, PATTERN-pair-teardown, PATTERN-durable-vs-ephemeral-state, PATTERN-lyxdirs-single-declarer, PATTERN-shed-run-directory: the loop's files (pid, log, envelope, trace copy, stderr) are declared by `internal/shedrun` under the run's ephemeral steps directory.
- PATTERN-test-speed and PATTERN-test-economy: the loop is tested at tier 1 through a seam for the child runner, the clock and the activity hook; one `integration` test runs a real child; `cmd/lyx/testdata/test-budget.yaml` rows rise where needed.
- `loom.yaml` gains `step_idle_timeout_min`; the hub reconcile seeds it on the binary change (PATTERN-binary-change-reconcile).
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
- The fork's context cost depends on the provider's cache behaviour; if a fork turns out costlier than reading the trace in place, the stencil falls back to reading in place for small traces.

## Acceptance criteria

- `--until-stop` runs steps until `continue` is false and prints one envelope whose step keys equal the stop step's short envelope and whose `loop` key is the closed set above.
- A stop condition `--before X` set during a run pauses the run with `current_producer` X before X runs; `--after X` pauses once X returned, at the routed row, for a Done and for a bounced Stuck; a halt of X leaves `--after X` recorded; `--clear` removes both and the bare request; an unknown target is refused listing the producers; the Go driver `lyx loom run` honours the same conditions.
- The pause reason names the condition and reaches the envelope.
- A child step that shows no activity for the window is killed with its process group, its stderr captured, and the loop stops with `kind: interrupted`, `loop.stop: interrupted`, the policy of the row and the status file written `failed`; a child that exits without an envelope stops the same way with cause `exited`.
- A pre-step refusal (missing hub config, unreadable seed) answers with `kind: bootstrap`, a trace file holding the refusal, and the status file written `failed` when one exists; the missing-config row's way forward is in the message.
- A busy refusal stops the loop and writes no status.
- A transient error is re-stepped once by the loop; a second failure at the same producer stops with both recorded on the envelope.
- The stop step's trace is copied into the steps directory and named by the envelope; a sweep that removes the original leaves the copy.
- A second `--until-stop` while a loop is alive spawns no loop and prints the same envelope when the loop ends; a loop that ended unwatched is delivered by the next invocation and then retired; the loop's children die with the loop and survive the waiter's death.
- The step envelope carries no `remaining` and no empty keys; the status envelope keeps `remaining`.
- The plan gate's informational WARN carries a count and a path, and the file holds the findings; the drift notice appears once per process.
- `pairteardown` kills a live loop before removing a worktree and tolerates an absent pid file.
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
