# Decision record — regression-baseline-verify

## Goal

Let a hub run a regression suite around a task's implementation: capture its profiles on the task's branch point before any code changes, compare after the implementation, and let a strong agent judge and fix the drift.
Regression is no test tier: no gate runs it, it is never narrowed or carried, and its outcome is a drift judgement, not pass or fail.
Three new producers, `Drift-Baseline`, `Drift-Await` and `Drift-Verify`, wired into loom and darn, carry it, and a hub that leaves the feature off sees none of them do anything.

## Scope

In:

- A new module owning the three producers, the hub-wide `drift` config module, the capture and compare runs and their records.
- A `lyx drift` CLI subtree for the agents and the operator: recording scope, accepting and withdrawing drift, deferring regression at a `Drift-Await` or `Drift-Baseline` halt, waiving it at a `Drift-Verify` halt with no counted capture (D5), running a compare or a side capture, and showing the capture's state.
- The capture's process plumbing in existing code (D4): a new `runproc` role `capture`; `agentActivityFor` reading the capture lock and the compare marker; on Windows the step and loop jobs admitting breakaway, the capture holding a named job through `proc.HoldNamedJob`, and `pairteardown` terminating that job through `proc.TerminateNamedJob`.
- Two agent stencils, for `Drift-Baseline` and `Drift-Verify`.
- Recipe wiring in `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`, the new row-name constants and the shedrecipe registry entries.
- A new gate key, `drift-verify`, in the gate template, for the test gate over `Drift-Verify`'s own code changes.
- Docs: the new package's `doc.go`, `docs/overview.md`'s module table and execution stack, and the doc lists of gate-slot holders, verify sites and run-process roles.

Out:

- Any behaviour change of an existing producer, Discussion's stencil included.
- A drift report format pinned in `contracts/specs/`: the agent reads the compare tool's own output.
- The sandbox hub's capture and compare harness, which belongs to board note `csharp-sandbox-hub`, where the Drift rows are proved end to end.
- The Models side: its hub config and a skip-file reader in its harness, which follow on OSL-1046, and the Models run there, which is the final confirmation.
- Issue #609's other wants: drift outcome and tier wall times in the change description, per-batch drift snapshots, the slow-suite tier.
- Turning the feature on in this hub.

## Decisions

### D1. A new module and the hub-wide `drift` config module

A new package owns the three producers' logic, the capture and compare runs, their records and the `drift` config module; its CLI is a `lyx drift` cobra subtree wired through `internal/cliwire`.
`drift` is registered in `internal/configreg` as `HubWide`, its template in the new package, loaded only through `configengine`.
Keys:

- `enabled` — bool, default `false`; the one switch turning regression on for the hub.
- `capture` and `compare` — each `{command, dir, env, timeout_min}`, `dir` resolved as a gate tier's `dir` is (default `.`), `timeout_min` at least 1; the template's `{}` declares none.
  Both are executable keys, read only from the template and the board default layer, as gate tiers are.
- `baseline_model` — model-spec of the `Drift-Baseline` agent, default `sonnet[medium]`.
- `verify_model` — model-spec of the `Drift-Verify` agent, default `opus[high]`.
- `verify_max_runs` — the budget of compare and side-capture runs one `Drift-Verify` agent session may make, default 8, at least 1.

A command renders two placeholders: `{baseline_dir}`, the absolute directory the capture writes and the compare reads, and `{skip_file}`, the absolute path of a file listing one test name per line, empty when nothing is skipped.
The config load refuses a declared `capture` or `compare` whose command lacks `{baseline_dir}`, so no capture writes where lyx cannot see it; `{skip_file}` is optional.
With `enabled` true, every Drift row and `lyx drift` verb that runs a command refuses on an undeclared `capture` or `compare`, naming `lyx config drift` as the way forward.

Rationale: the operator's design puts regression in its own config block of the module owning the Drift rows; the rows run in two recipes, so the block cannot live in `loom.yaml` or `darn.yaml`.
A file of test names is the one skip interface any harness can read.

### D2. Recipe positions and the off switch

Loom: `Discussion-Bouncer` → `Drift-Baseline` → `Plan-Write`; `Plan-Bouncer` → `Drift-Await` → `Webster`; `Code-Bouncer` → `Drift-Verify` → `Describe`.
Darn: `Preflight` → `Drift-Baseline` → `Darn-Write` → `Drift-Verify` → `Code-Bouncer`; darn has no `Drift-Await`, and darn's Code perch fixes after `Drift-Verify` go uncompared, as the entry orders.
A Finalize landing's merge and merge fixer go uncompared too, and so does darn's Finalize reroute to `Code-Bouncer`.
A loom Finalize reroute re-enters `Drift-Verify` through `Parent-Align` and the Code perch and compares again, keeping the scope record's acceptances, unless regression was waived (D6).
None of the three rows carries an `on_stuck`, so a Stuck halts the run through the shed stop chain: rule table, tackler, arbiter, then the parent.
The arbiter may settle a Drift halt that is no decision halt with the way forward the halt names, `lyx drift accept` included under scope `partial`.
Under scope `full`, every `Drift-Verify` halt but an error or transient stop is a decision halt for the parent (D6), which the arbiter never settles, and a caller's `lyx drift accept` refuses at any other halt (D7), so no drift in a task meant to change nothing is accepted before the parent rules.

`Drift-Baseline`, `Drift-Await` and `Drift-Verify` all carry `entry_gate: drift.enabled`, per the parent's ruling below, so a hub with the flag off records them `Skipped` and spawns or runs nothing, with no recipe overlay.
The gate is evaluated when the run enters the row: `shedengine.Shed.enter` skips the evaluation while the status's `EnteredRow` names the row, which a halt at the row leaves set and `lyx loom goto` sets to its target.
So a flag turned off after the run entered a Drift row takes effect at the next entry of a Drift row, never at that one (D5).

`Drift-Baseline` follows the Discussion perch, so it reads the approved decision record.
`Drift-Verify` follows the Code perch, so it compares the code the perch left; since `Parent-Align` runs before the Code perch, the compared tree carries the parent's commits merged since the baseline (D7).

### D3. `Drift-Baseline`: scope decision, then capture

The row spawns one agent on `drift.baseline_model`.
In loom it reads the decision record; in darn, set by the row's config key `subject: board-entry`, it reads the task's board entry.
The agent records the task's regression scope with `lyx drift scope --scope off|partial|full [--skip <test>]...`, which writes the Go-owned scope record `_lyx/drift/scope.json` through `internal/state`:

- `off` — the task changes the model on purpose; nothing is captured or compared.
- `partial` — the tests named by `--skip` will drift totally and are run neither at capture nor at compare.
- `full` — a pure refactoring; every test must stay byte-identical.

It writes its reasoning and the drift it expects to `_lyx/drift/baseline.md`, freeform; the row's outputs are both files.
`lyx drift scope` refuses unless the run's current row is `Drift-Baseline`, refuses once a scope record exists, refuses `--skip` with any scope but `partial`, and refuses `partial` with no `--skip`.

With scope `off` the row returns Done.
Otherwise it starts the capture (D4): in loom detached, returning Done once the child holds the capture lock, so Plan-Write runs meanwhile; in darn, set by the row's config key `capture: wait`, in the foreground, returning Done on a passed capture and Stuck on a failed one.
A foreground capture's Stuck on a failed capture, in darn and in loom's Windows fallback (D4), names `lyx drift capture` followed by `lyx loom resume` to capture again, or `lyx drift defer` followed by `lyx loom resume` to go on with regression deferred (D5).
The row commits `scope.json` and `baseline.md` with the run's records through a records commit seam before it returns.

On re-entry a present scope record skips the agent only when `baseline.md` exists as well; otherwise the agent is spawned to write `baseline.md` alone, against the recorded scope.
With `awaited_at` set (D5) the row returns Done without a capture.
Otherwise an absent capture record counts as a capture that did not pass, and a `running` record whose lock is free counts as failed.
A capture holding the lock is waited on in darn, as the row's own foreground capture is, and left running in loom.
A capture is started again only when none holds the capture lock and the last one did not pass; a passed capture is left alone.
A capture D4 refuses, because HEAD left the branch point or the tracked tree is dirty, ends the row Stuck naming `lyx drift defer` followed by `lyx loom resume`; on a dirty tree with HEAD still the branch point it also names committing or cleaning the tree and `lyx loom resume`, which captures again.
Every `Drift-Baseline` Stuck on a failed or refused capture has a reason opening with a fixed capture-halt marker, which `lyx drift defer` reads to admit that halt.
In loom a recapture after a `Drift-Await` halt goes through `lyx drift capture` and `lyx loom resume` (D5), never `lyx loom goto Drift-Baseline`: the goto goes on to `Plan-Write`, whose fresh spawn archives the approved plan, so the whole Plan perch would run again.

Rationale: a Sonnet agent is enough to read a design and name the tests it moves; the verb keeps the scope a Go-written record, so no agent-written format needs a parser.

### D4. The capture run

The capture is a detached `lyx drift capture` child of the row, registered with `runproc.ChildRecord` under a new role `capture` and logged as a spawn through `internal/logger`.
It holds an OS lock on `.lyx/drift/capture.lock` for its whole life; a second capture refuses while the lock is held, and a released lock is how every reader tells a dead capture, so no pid check or pid reuse is involved.
A reader probes the lock with a non-blocking take it releases at once, and a capture start retries its own non-blocking take for a few seconds before refusing, so a reader's probe never makes a capture refuse.
A main capture refuses unless the run's current row is `Drift-Baseline`, or `Drift-Await` while the run is halted there, and HEAD is the task's branch point, its merge base with the parent branch, and refuses a worktree whose tracked files are dirty; so no capture ever runs on changed code, and the backup below is the only refresh, reachable only before the first code change.
Run by a caller at a `Drift-Await` halt, or at a `Drift-Baseline` halt on a failed or refused capture, `lyx drift capture` starts the same detached, registered child the row starts and returns once the child holds the capture lock, so the capture outlives the caller's own process and ends with the run's processes; a resumed `Drift-Baseline` in darn waits on it (D3).
The task worktree's `.lyx/` is already excluded through `.git/info/exclude` at clone, so the baseline directory is never dirt.

On Windows a step runs in the unnamed kill-on-close job `proc.StartTied` gives it, nested in its loop's named job, and every process left in that job dies when the step exits.
So the row and `lyx drift capture` start the capture child with `proc.DetachBreakaway`, and `proc.StartTied`'s job and the loop's named job gain `JOB_OBJECT_LIMIT_BREAKAWAY_OK`, without which Windows refuses that start; a process started without the breakaway flag stays in its job, so every descendant of a step that asks for no breakaway still dies with it.
`spawnLoop` in `internal/shedverbs/loopwait.go`, the one other non-test `proc.DetachBreakaway` caller, is reached only from the `--until-stop` waiter's spawn branch, taken when the run's loop lock is free.
No step child runs that waiter: every waiter runs in a driver, a run verb or the arbiter's relaunch line, outside any step, and a step's own run holds its loop lock, so a waiter there would follow the loop and spawn none.
A loop is meant to outlive its waiter, so one a future step started would leave the step's job as intended; the plan needs no check.
The capture holds a named kill-on-close job of its own through `proc.HoldNamedJob`, named from the worktree as `shedrun.LoopJobName` names a loop's job, and `pairteardown`, the adopt's end of a local pair included, terminates that job through `proc.TerminateNamedJob`, as it ends a loop on Windows.
A job name lives in one machine's object namespace, so this reaches only a capture of this machine's own pair.
When a job lyx did not create still refuses the breakaway, the row runs the capture in the foreground as darn does, returning Done on a passed capture and Stuck on a failed one with darn's ways forward (D3), and `lyx drift capture` falls back to `proc.Detach` as `spawnLoop` does, so its capture lives as long as the caller's shell job.

While the capture lock is held, and while a Drift row's own mechanical compare runs, the run's activity reading counts as activity now.
`agentActivityFor` in `internal/loomcli/looparm.go` reads the capture lock and a compare marker under `.lyx/drift/`, as it reads a live parent-lock wait, so the loop's step idle watchdog (`step_idle_timeout_min`) never kills `Drift-Await`'s wait, darn's foreground capture or a mechanical compare.
The compare marker is an OS lock the compare takes before it waits for its gate slot and holds until it ends, probed as the capture lock is: a marker left by a dead compare reads as none, and a compare queued for a slot reads as activity.
A capture holding the lock while another row's agent hangs keeps that watchdog off too, until the capture ends: its timeout counts from the command's start (below), so a capture queued for a gate slot keeps every watchdog off until it gets one, and for at most `capture.timeout_min` after.

It holds one `internal/gateslot` slot as a non-tier holder for the command's run, so it runs within the hub's slot count, and the plan gate's type load, also a slot holder, may queue behind it.
It renders `capture.command` with `{baseline_dir}` set to the baseline directory under the worktree's `.lyx/drift/` and runs it in `capture.dir` with `capture.env`.
A capture that finds a non-empty baseline directory renames it to a numbered backup beside it first, so no run deletes or overwrites a baseline.

The capture record `.lyx/drift/capture.json`, written through `internal/state`, holds the state (`running`, `passed`, `failed`), HEAD's commit, the slot-acquired and command-start times, the end time, the exit code, the reason and the log path; a fresh capture writes a fresh record.
Passed means exit code 0 and at least one file written to the baseline directory.
`capture.timeout_min` counts from the command's start on Go's monotonic clock, which on Linux does not advance while the machine sleeps.
The capture starts its command through `internal/proc`'s `StartTied` and at the timeout kills the command's whole tree through `Tied.Kill`, a process group on Linux and a kill-on-close job object on Windows, as the gate's `runTiedChild` (`internal/gatecli/child.go`) ends a timed-out test run; it records `failed` once the command is gone, then releases the slot and the lock.
The command is an unregistered child of the capture process, so ending it touches no `runproc` record.
The command's combined output goes to a log under `_lyx/drift/logs/`, one file per capture, compare or side-capture run, the open log file itself being the command's stdout and stderr, so no copying pipe a descendant that escaped the tree kill holds open keeps the capture from recording `failed` and releasing its lock; the row that next ends after the run commits it with the run's records: in loom `Drift-Await` the capture log, on every way out it takes, Done, Stuck and a deferred Done alike; in darn `Drift-Baseline`; and `Drift-Verify` its own runs' logs and any capture log still uncommitted.

Rationale: issue #609 requires the capture at the branch point, in the task worktree, before the first code change, and never on half-implemented code.

### D5. `Drift-Await`: hold Webster until the baseline exists

A mechanical row, no LLM, reading the scope and capture records, each read again after a lock check so a capture that just ended is read as ended:

- no scope record, or scope `off` — Done.
- scope `awaited_at` set (below), by a passed await or a deferral — Done whatever HEAD is.
- scope `partial` or `full` with no capture record — Stuck.
- capture `running` with its lock held — wait, waking at most once a second and backing off while idle, with no bound of its own: the capture enforces its own timeout (D4) and releases the lock on exit, so the record is read once the lock is free; the held lock reads as activity to the loop's step idle watchdog (D4), so the wait is never killed as idle.
- capture `running` with its lock free, or `failed` — Stuck naming the log.
- capture `passed` — Done when HEAD is still the capture's commit, writing `awaited_at` into the durable scope record and committing it with the capture log; Stuck when HEAD moved.

A Stuck while HEAD is still the branch point names `lyx drift capture` followed by `lyx loom resume`, which captures again on the branch point and re-enters `Drift-Await` (D4), or `lyx drift defer` followed by `lyx loom resume` to go on with the regression question deferred.
A Stuck after HEAD moved names only `lyx drift defer` followed by `lyx loom resume`, since no capture may run on changed code.
`lyx drift defer` writes `awaited_at` into the scope record, marked as a deferral, and refuses unless the run is halted at `Drift-Await`, or at `Drift-Baseline` on a failed or refused capture (D3); `Drift-Baseline` then returns Done on resume.
It also refuses while a capture holds the capture lock, naming `lyx loom resume`, whose row waits on the running capture, so no capture still running when a deferral lets code change ever becomes the counted capture (below).
After a deferral past a failed, missing or refused capture, `Drift-Verify` halts a second time, naming `lyx drift waive` followed by `lyx loom resume` (D6); after a deferral past a HEAD-moved halt the passed capture still exists and, with `awaited_at` set, is the counted capture (below), so `Drift-Verify` compares against it.
`awaited_at` in the durable scope record lets a later entry, after Webster's reroute through the plan perch, pass without a baseline check, a deferred one included.

The capture `Drift-Verify` counts, the counted capture, is a capture record in state `passed` that, in loom, has `awaited_at` set in the scope record; in darn, which has no `Drift-Await`, a `passed` record alone.
So in loom a capture no `Drift-Await` awaited or deferred past is never compared against, and `Drift-Verify` halts on no counted capture (D6).

The flag is read only when the run enters a Drift row (D2).
Turned off before the run enters `Drift-Await`, it records `Drift-Await` Skipped, and `Drift-Verify` too while it stays off, so a capture left running is ignored and ends at its own timeout or with the run's processes; turned back on before `Drift-Verify` is entered, `Drift-Verify` finds no counted capture and halts.
Turned off while `Drift-Await` waits or is halted, it does not skip that row: the wait goes on until the capture ends, and a resume from the halt runs the row again.
Turned off while `Drift-Verify` is halted, it does not skip that row on resume either.
At an entered row the way to give up regression is the halt's own: `lyx drift defer` at a `Drift-Await` halt, after which `Drift-Verify` records Skipped when the run enters it with the flag still off; `lyx drift waive` at a `Drift-Verify` halt on no counted capture; and at any other `Drift-Verify` halt none, since that halt's compare can run, so only the ways forward it names lift it.
Turning the flag off mid-run is the operator's choice to give up regression for that run, Webster building beside a running capture included.

An adopt on another machine carries the scope record but not `.lyx/drift/`, which is never tracked, so the adopting machine has no capture record or baseline.
The adopt reaches no process of the other machine: a capture still running there ends at its own timeout or with that machine's teardown of its pair.
In loom, before `awaited_at` is set, `Drift-Await` halts on the missing capture record, and on the branch point `lyx drift capture` with `lyx loom resume` captures there; once `awaited_at` is set, `Drift-Await` passes and `Drift-Verify` halts on the missing capture, naming `lyx drift waive` followed by `lyx loom resume`, since no capture may run on changed code.
In darn, an adopt while `Drift-Baseline` is the current row re-enters it and captures again (D3); a later adopt leaves `Drift-Verify` to halt on the missing capture, naming the same waive and resume (D6).

Rationale: no code may change before the baseline exists, and a failed capture never falls back to one taken later.

### D6. `Drift-Verify`: compare, then one strong agent on drift

The row reads the scope record: no record or scope `off` returns Done with that reason.
A scope with no counted capture (D5) halts Stuck, naming `lyx drift waive --reason <text>` followed by `lyx loom resume` as the way to go on without regression.
`lyx drift waive` writes a waived mark with its reason into the scope record, and on every later entry `Drift-Verify` returns Done without a compare.
It lets the whole regression question through, so it refuses unless the run is halted at `Drift-Verify` with no counted capture, naming the ways forward the halt itself names at any other `Drift-Verify` halt.
The row's agent, spawned only after a compare, never reaches it, and under every scope the halt on no counted capture is a decision halt (below), so only the parent waives.
Otherwise it runs the compare mechanically, holding one gate slot and ended at `compare.timeout_min` as D4 ends a capture, with `{skip_file}` listing the scope's skipped tests and the accepted ones (D7).
A compare passes on exit code 0.
A passing compare returns Done and spawns nothing.

A failing compare spawns one agent on `drift.verify_model`, given the compare's log, the scope record, `baseline.md`, the decision record as amended (loom) or board entry (darn), and the baseline's commit.
Within its budget it judges which drift is acceptable against the task and `baseline.md`'s expectation, accepts that drift (D7), finds the cause of the rest and fixes it, committing each fix, until the compare passes.
An amendment after Discussion never re-scopes; the agent judges drift against the amended record.
It runs regression only through `lyx drift compare` and `lyx drift capture --side`, whose runs count against `drift.verify_max_runs` in a Go-written counter; past the budget both verbs refuse, naming the report and the stop as the way forward.
Both verbs refuse unless the run's current row is `Drift-Verify`, naming that row as the place they run.
The row resets the counter at each fresh spawn of its agent, so each attempt has `verify_max_runs` runs; the row's own mechanical compares never count.
Each resume that reaches a failing compare, a tackler's or the arbiter's included, spawns a fresh agent, so the number of attempts is bounded by the run's `tackler_max_spawns` and `arbiter_max_spawns` spawn budgets in `shed.yaml` (`internal/sheddriver`), counted per run, past which every stop hands back to the parent.
A side capture writes to a fresh numbered directory under `.lyx/drift/side/`, renders `{baseline_dir}` as that directory, needs no clean tree or branch point, and writes neither the capture record nor the baseline directory.
The agent writes its reasoning, every acceptance and every fix to `_lyx/drift/verify-report.md`, the row's output.
It stops without passing the compare when it is unsure or past its budget.

When the agent ends, the row's handoff runs in this order on every attempt: a dirty tree re-prompts the session to commit; then the `verify` gate over the hub's new `drift-verify` gate key (scope `diff`) through `verifytree.Verify`, when the session committed code; then the compare, last, so no fix the agent or a gate re-prompt makes goes uncompared.
A dirty tree or a failing test gate re-prompts the session up to three times, then is Stuck.
A failing final compare is Stuck at once, naming the report and the ways forward: `lyx drift accept` and resume, or fix and resume.
Under scope `full` each acceptance the agent session makes is marked `unruled` in the scope record, and the parent's own at a decision halt is `ruled` (D7).
The row checks the record at every entry, before any compare, and when its agent ends, after the handoff above: under scope `full` a record holding an `unruled` acceptance ends the row Stuck in a decision halt, even on a passing compare, so the parent rules on drift in a task meant to change nothing.
Every decision halt under scope `full` reached while an `unruled` acceptance stands, this one or another below, names those acceptances and the parent's ways forward for them beside its own: `lyx loom resume` to stand by the acceptances, after any `lyx drift accept` of the parent's own, or `lyx drift withdraw` and resume to reject one.
As the row returns such a halt, it records in the scope record the `unruled` acceptances it names and the time.
An entry is the resume from that halt only when the run status's last history entry is a Stuck of this row whose latest time, `LastAt` when shed folded repeated identical Stucks into it and `At` otherwise, is at or after that time; only then does the row mark the recorded acceptances that survive `ruled`, before it compares, and a passing compare then returns Done with no agent.
So an agent session that accepts and then ends in an error, transient or interrupted stop leaves an `unruled` acceptance no halt recorded, and the resume of that stop, a tackler's included, halts on entry for the parent instead of comparing.
Under scope `full` every `Drift-Verify` halt has a reason opening with `shedadapters.DecisionHaltPrefix`: this one, a failing final compare, the halt on no counted capture, and the dirty-tree and test-gate Stucks after their re-prompts.
So `arbiterstop.HandsBack` marks it and the stop chain hands it only to the parent, ahead of the tackler and the arbiter, which makes the resume from it the parent's.
An error or transient stop of the row is the one exception: it is resumed, never settled with an acceptance (D7), and an `unruled` acceptance it leaves is put to the parent at the resumed entry.
Under scope `partial` the halt on no counted capture has that prefix too, so no arbiter gives up regression for a run, and so does a failing final compare while the scope record's withdrawn list is non-empty (D7).
The row commits the report, the scope record and its logs with the run's records on every way out.
No review follows the agent's fixes, and the agent is no perch.

Rationale: small unwanted drift is beyond a Sonnet implementer and takes a strong agent with tracing and many varied runs; one agent both judging and fixing is the operator's design.

### D7. Accepting drift without touching the baseline

`lyx drift accept <test> --reason <text>` appends the test and reason to the scope record's accepted list; every later compare skips it.
Each accepted entry carries who accepted it: the row's live agent session or a caller at a `Drift-Verify` halt.
Under scope `full` it also carries its ruling: `unruled` for the agent session's, until the parent's resume from a decision halt marks it `ruled` (D6), and `ruled` for a caller's, which only the parent makes there.
`lyx drift withdraw <test>` moves an accepted entry to the scope record's withdrawn list, so the next compare runs the test again.
The agent session's `lyx drift accept` refuses a withdrawn test, naming the parent's ruling, and a caller's refuses one unless the run is halted at a decision halt, which only the parent lifts, naming the parent's ruling as the way forward; so a rejection holds across resumes and against the arbiter.
Both verbs refuse unless the run's current row is `Drift-Verify`, which a Drift-Verify halt keeps as the current row.
Under scope `full`, a caller's `lyx drift accept` refuses unless the run is halted at a decision halt, naming the parent's ruling as the way forward; every `Drift-Verify` halt under `full` but an error or transient stop is one (D6), so only the parent accepts there.
So the row's agent, the parent or operator at a halt, and under scope `partial` the arbiter at a halt that is no decision halt, are the callers that accept or withdraw.

Under scope `partial` an acceptance can let through up to every test the compare runs, a compare left with no test included, and the row then returns Done with no review.
The agent's judgement, against the task and `baseline.md`'s expected drift, is the only bound on what it accepts, and the arbiter may accept under `partial` at a halt that names `lyx drift accept` and is no decision halt; every acceptance and its reason stands in `verify-report.md` and the committed scope record.
Under scope `full` the agent's acceptances stay `unruled` until the parent rules, and the row ends in a decision halt at every entry and attempt end while one stands, however the attempt ended (D6), so the parent bounds them.

Drift the agent traces to a parent commit merged after the baseline's commit is acceptable, and the report names the commit.
Nothing in this design deletes, overwrites or refreshes a baseline after the first code change.

Rationale: an accepted drift must let the compare pass without blessing it in the baseline, and the test is the granularity both the harness and the skip file speak.

### Added after Discussion (parent, 2026-10-25): Drift-Verify sits after the Code perch, before Describe

Decision: In loom, Drift-Verify runs after Code-Bouncer and before Describe, so it compares the code as it lands: the task's changes, the Code perch's fixes and the parent commits Parent-Align merged in. Drift caused by merged parent commits is judged under D7's parent-commit acceptance. Answers review finding F9.

Rationale: Operator ruling, 2026-10-25. Since fc75aef79 Parent-Align runs before the Code perch, so the entry's placement before Parent-Align is no longer possible.

### Added after Discussion (parent, 2026-10-25): Drift-Await is skipped through its entry gate when regression is off

Decision: Drift-Await gets entry_gate: drift.enabled like the other Drift rows, and with the flag off it records Skipped and runs nothing. Every producer's entry gate is the boolean check for whether it runs at all; no special case keeps Drift-Await ungated. Answers review finding F10.

Rationale: Operator ruling, 2026-10-25: with the flag off the Drift rows skip, and the entry gate is the mechanism for that.

### Added after Discussion (parent, 2026-10-25): Drift-Verify's dirty check reads the code checkout's tracked paths only

Decision: driftDeps' dirty-paths seam is fabricengine.SourceDirtyPaths(location), the code worktree's tracked dirt, never Fabric.DirtyPaths, which also lists the records checkout. In 11-drift-verify-producer.md step 7 and its dirty gate entry, the dirty tree is the code checkout's tracked paths, and records-side paths such as _lyx/drift/scope.json never count. TestDriftVerify gains a row with only a records-side path dirty, asserting no dirty Stuck.

Rationale: An agent's lyx drift accept and steps 2 and 3 write records-side files before the check, so counting the records checkout would make every run Stuck; SourceDirtyPaths already reads only the code worktree's tracked paths.

## Constraints

- The design departs from no PATTERN entry.
  `_lyx/drift/` holds the scope record, the two reports and the logs; `.lyx/drift/` holds the capture record and lock, the compare marker, the baseline and side directories, the skip file and the run counter; only `internal/lyxdirs` names them.
  Every refusal and halt names its way forward and a test reaches each; `lyx drift` verbs stay allowed to agents.
  Both stencils are read through `internal/stencilstore` and render the parent and edit directives; both models go through `NewModelChoice`.
- The capture runs in the task worktree, never in a separate checkout.
- Between `Drift-Baseline` and `Drift-Await` no row's own code builds in the task worktree, and the plan gate's type load is Go-only; a Plan reviewer's `lyx gate test`, which its tool-use rules ask for, runs beside the capture within the hub's slot count, and its plain build beside it outside any slot.
- Models (issue #609, 2026-10): the project root is `src/csharp/NORCE.Models/`, not the git root, hence `dir`; compare mode is `-e NORCE_BASELINE_MODE=verify`, where a missing baseline fails; `NORCE_BASELINE_DIR` sets the capture directory.
- `pairteardown` ends a pair's processes, the capture included: `runproc.EndSparing` ends the capture on Linux, and its named job ends it on Windows (D4).

## Auto-mode assumptions

- The default models and budget in D1, the skip file in D1, the log location in D4, D6's own `drift-verify` test gate over the agent's fixes, D5's `lyx drift defer`, D6's `lyx drift waive` and D7's `lyx drift withdraw` are auto-picks.

## Open risks

- In a .NET hub a Plan reviewer's build shares `bin` and `obj` with the capture's `dotnet test` in the same worktree; board note `csharp-sandbox-hub` proves whether that breaks a capture.
- Models-side facts this machine cannot check, for OSL-1046: whether the harness reads `NORCE_BASELINE_DIR` in verify mode, whether its all-`Inconclusive` capture exits 0 under the `dotnet test` runner it uses, and whether its compare fails when it discovers no test; the harness also reads no skip file yet.
  D4's pass rule asks for written files as well as exit 0, so a capture that ran nothing fails either way.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - flag-off-gate-only-at-row-entry (class consistency, severity MEDIUM, open in rounds 3)
  - scope-full-unruled-acceptance-survives-interrupt (class design, severity BLOCKING, open in rounds 3)
  - waive-precondition-contradicts-halt (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - dirty-read-checkout-unspecified (class decision, severity MEDIUM, open in rounds 3)
  - gate-halt-cleared-before-gate-runs (class design, severity MEDIUM, open in rounds 3)
  - resume-test-time-format-mismatch (class design, severity BLOCKING, open in rounds 3)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - compare-marker-no-retry (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- With `drift.enabled` false, a loom and a darn run record every Drift row they carry as `Skipped`, and neither spawns an agent or runs a command.
- Turning the flag off with a capture running before the run enters `Drift-Await` skips `Drift-Await` and `Drift-Verify`; turned back on before `Drift-Verify` is entered in loom, the row halts on no counted capture, compares nothing, and `lyx drift waive` there is admitted.
- Turning the flag off while `Drift-Await` waits or is halted, or while `Drift-Verify` is halted, skips neither row: the wait goes on and a resume runs the halted row.
- With the flag on, an undeclared `capture` or `compare` is refused naming `lyx config drift`, and a command without `{baseline_dir}` is refused at load.
- Loom: the capture starts detached and `Drift-Baseline` returns before it finishes; `Drift-Await` waits on a running capture while its lock is held, with no bound of its own, a capture waiting for a slot included, and passes when it ends passed; a capture longer than `step_idle_timeout_min` is waited out without a watchdog kill.
- A capture started by the row outlives the row's step, on Windows too, and teardown ends it.
- Darn: `Drift-Baseline` returns only after the capture passed, and is Stuck on a failed one, naming `lyx drift capture` and `lyx drift defer`; after `lyx drift defer` and `lyx loom resume` it returns Done; re-entered after its process died mid-capture, it captures again.
- Re-entered with a scope record but no `baseline.md`, `Drift-Baseline` spawns its agent, whose `lyx drift scope` refuses; re-entered off the branch point with `awaited_at` unset, it is Stuck naming `lyx drift defer`; on Windows with breakaway refused, loom's foreground capture is Stuck on a failure naming `lyx drift capture` and `lyx drift defer`.
- Scope `off` captures and compares nothing; `partial` writes the skip file and passes it to capture and compare; `lyx drift scope` refuses the invalid combinations and outside `Drift-Baseline`.
- A failed capture, a timed-out capture whose tree was killed and whose record says `failed`, a capture whose lock is free while its record says `running`, a missing capture record under a non-off scope, and HEAD moved before the first await each halt `Drift-Await`, naming the way forward D5 gives for the case.
- At a `Drift-Await` halt on the branch point, `lyx drift capture` after a failed or dead capture captures again and `lyx loom resume` then passes `Drift-Await` without re-running `Plan-Write`; a main capture refuses off the branch point, outside `Drift-Baseline` and a halted `Drift-Await`, and on a dirty tree.
- `lyx drift defer` refuses unless the run is halted at `Drift-Await`, or at `Drift-Baseline` on a failed or refused capture, and refuses while a capture holds the capture lock; after it and `lyx loom resume` past a failed or missing capture, `Drift-Await` returns Done on every later entry and `Drift-Verify` halts naming `lyx drift waive`.
- `lyx drift waive` refuses unless the run is halted at `Drift-Verify` with no counted capture, and `lyx loom resume` after it reaches Done without a compare; that halt is a decision halt under every scope.
- A capture probe by a reader while a capture starts never makes the capture refuse.
- A capture started by `lyx drift capture` at a `Drift-Await` halt keeps running after the caller's process exits.
- After an adopt on another machine in loom, with `awaited_at` unset `Drift-Await` halts on the missing capture record, and with it set `Drift-Verify` halts naming `lyx drift waive`; in darn, `Drift-Verify` halts naming `lyx drift waive` too.
- A capture exiting 0 with no file written fails; an existing baseline directory is moved to a backup, never overwritten.
- After `awaited_at` is set, `Drift-Await` re-entered after Webster committed code returns Done.
- Capture and compare each hold a gate slot and wait while every slot is taken.
- A passing compare in `Drift-Verify` spawns no agent; a failing one spawns one, and the row is Stuck when the final compare still fails.
- A test fix after a gate re-prompt is compared again before Done, and a dirty tree after the session is re-prompted.
- Under scope `full`, an acceptance the agent session made ends `Drift-Verify` Stuck in a decision halt the arbiter never settles, and the parent's accept and resume then reaches Done on a passing compare.
- Under scope `full`, an agent session that accepts and then ends in an error, transient or interrupted stop does not reach Done on resume, a tackler's included, until the parent has ruled: the resumed entry halts in a decision halt before any compare.
- Under scope `full`, every `Drift-Verify` halt but an error or transient stop is a decision halt, and a caller's `lyx drift accept` refuses at a halt that is no decision halt.
- `lyx drift compare` and `lyx drift capture --side` refuse past `verify_max_runs` and outside `Drift-Verify`, and a fresh agent spawn resets their counter; a side capture leaves the baseline directory and capture record untouched; `lyx drift accept` and `lyx drift withdraw` refuse outside `Drift-Verify`, an accepted test is skipped by the next compare, and a withdrawn one is run again, refused to the agent session's `lyx drift accept`, and refused to a caller's outside a decision halt; under scope `partial` a failing final compare with a non-empty withdrawn list is a decision halt.
- The scope record, reports and logs are committed with the run's records.
- Tests use fake capture and compare commands; none needs Models.

## Notes for the plan writer

- The new rows need `Name*` constants beside loom's and darn's existing ones for the coverage guard.
- `shedadapters.DecisionHaltPrefix`'s doc names only the bouncer's decision halt; D6's `Drift-Verify` decision halts widen its meaning, so its doc, or a shared hand-back marker it moves to, covers them.
- `Env.SettingKnown`, which an entry gate on a `module.key` operand needs, is filled in `internal/loomcli/wiring.go`; darn's build at `internal/loomcli/arm.go` uses the same env, so it is already filled.
- The C# sandbox hub at `/home/knatte/Code/lyx-csharp-LYXHUB` is where board note `csharp-sandbox-hub` proves the Drift rows end to end with its own harness, outside this task; the Models run on OSL-1046 is the final confirmation.
