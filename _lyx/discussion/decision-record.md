# Decision record — regression-baseline-verify

## Goal

Let a hub run a regression suite around a task's implementation: capture its profiles on the task's branch point before any code changes, compare after the implementation, and let a strong agent judge and fix the drift.
Regression is no test tier: no gate runs it, it is never narrowed or carried, and its outcome is a drift judgement, not pass or fail.
Three new producers, `Drift-Baseline`, `Drift-Await` and `Drift-Verify`, wired into loom and darn, carry it, and a hub that leaves the feature off sees none of them do anything.

## Scope

In:

- A new module owning the three producers, the hub-wide `drift` config module, the capture and compare runs and their records.
- A `lyx drift` CLI subtree for the agents and the operator: recording scope, accepting and withdrawing drift, deferring regression at a `Drift-Await` halt, running a compare or a side capture, and showing the capture's state.
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
Changes made after `Drift-Verify` by Finalize's merge and merge fixer, and in darn by a Finalize reroute to `Code-Bouncer`, go uncompared too.
None of the three rows carries an `on_stuck`, so a Stuck halts the run through the shed stop chain: rule table, tackler, arbiter, then the parent.
The arbiter may settle a Drift halt with the way forward the halt names, `lyx drift accept` included under scope `partial`.
Under scope `full`, every `Drift-Verify` halt over drift is a decision halt for the parent (D6), which the arbiter never settles, so no drift in a task meant to change nothing is accepted before the parent rules.

`Drift-Baseline` and `Drift-Verify` carry `entry_gate: drift.enabled`, so a hub with the flag off records them `Skipped` and spawns nothing, with no recipe overlay.
`Drift-Await` departs from the operator's "skipped like the PR rows" and carries no entry gate: it acts on the run's own records (D5), never on config, so a capture already running when the flag is turned off still holds Webster back.
With no scope record it returns Done at once and spawns nothing, so with the flag off its history shows Done, not Skipped.

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
`lyx drift scope` refuses unless the run's current row is `Drift-Baseline`, refuses `--skip` with any scope but `partial`, and refuses `partial` with no `--skip`.

With scope `off` the row returns Done.
Otherwise it starts the capture (D4): in loom detached, returning Done once the child holds the capture lock, so Plan-Write runs meanwhile; in darn, set by the row's config key `capture: wait`, in the foreground, returning Done on a passed capture and Stuck on a failed one.
The row commits `scope.json` and `baseline.md` with the run's records through a records commit seam before it returns.

On re-entry a present scope record skips the agent.
A capture is started again only when none holds the capture lock and the last one did not pass: a `running` record whose lock is free counts as failed.
A passed capture is left alone.
In loom a recapture after a `Drift-Await` halt goes through `lyx drift capture` and `lyx loom resume` (D5), never `lyx loom goto Drift-Baseline`: the goto goes on to `Plan-Write`, whose fresh spawn archives the approved plan, so the whole Plan perch would run again.

Rationale: a Sonnet agent is enough to read a design and name the tests it moves; the verb keeps the scope a Go-written record, so no agent-written format needs a parser.

### D4. The capture run

The capture is a detached `lyx drift capture` child of the row, registered with `runproc.ChildRecord` under a new role `capture` and logged as a spawn through `internal/logger`.
It holds an OS lock on `.lyx/drift/capture.lock` for its whole life; a second capture refuses while the lock is held, and a released lock is how every reader tells a dead capture, so no pid check or pid reuse is involved.
A main capture refuses unless the run's current row is `Drift-Baseline`, or `Drift-Await` while the run is halted there, and HEAD is the task's branch point, its merge base with the parent branch, and refuses a worktree whose tracked files are dirty; so no capture ever runs on changed code, and the backup below is the only refresh, reachable only before the first code change.
Run by a caller at a `Drift-Await` halt, `lyx drift capture` starts the same detached, registered child the row starts and returns once the child holds the capture lock, so the capture outlives the caller's own process and ends with the run's processes.
The task worktree's `.lyx/` is already excluded through `.git/info/exclude` at clone, so the baseline directory is never dirt.

It holds one `internal/gateslot` slot as a non-tier holder for the command's run, so it runs within the hub's slot count, and the plan gate's type load, also a slot holder, may queue behind it.
It renders `capture.command` with `{baseline_dir}` set to the baseline directory under the worktree's `.lyx/drift/` and runs it in `capture.dir` with `capture.env`.
A capture that finds a non-empty baseline directory renames it to a numbered backup beside it first, so no run deletes or overwrites a baseline.

The capture record `.lyx/drift/capture.json`, written through `internal/state`, holds the state (`running`, `passed`, `failed`), HEAD's commit, the slot-acquired and command-start times, the end time, the exit code, the reason and the log path; a fresh capture writes a fresh record.
Passed means exit code 0 and at least one file written to the baseline directory.
`capture.timeout_min` counts from the command's start on Go's monotonic clock, which on Linux does not advance while the machine sleeps.
The capture starts its command through `internal/proc`'s `StartTied` and at the timeout kills the command's whole tree through `Tied.Kill`, a process group on Linux and a kill-on-close job object on Windows, as the gate's `runTiedChild` (`internal/gatecli/child.go`) ends a timed-out test run; it records `failed` once the command is gone, then releases the slot and the lock.
The command is an unregistered child of the capture process, so ending it touches no `runproc` record.
The command's combined output goes to a log under `_lyx/drift/logs/`, one file per capture, compare or side-capture run, the open log file itself being the command's stdout and stderr, so no copying pipe a descendant that escaped the tree kill holds open keeps the capture from recording `failed` and releasing its lock; the row that next ends after the run commits it with the run's records: `Drift-Await` the capture log in loom, `Drift-Baseline` in darn, and `Drift-Verify` its own runs' logs.

Rationale: issue #609 requires the capture at the branch point, in the task worktree, before the first code change, and never on half-implemented code.

### D5. `Drift-Await`: hold Webster until the baseline exists

A mechanical row, no LLM, reading the scope and capture records, each read again after a lock check so a capture that just ended is read as ended:

- no scope record, or scope `off` — Done.
- scope `awaited_at` set (below), by a passed await or a deferral — Done whatever HEAD is.
- scope `partial` or `full` with no capture record — Stuck.
- capture `running` with its lock held — wait, waking at most once a second and backing off while idle, with no bound of its own: the capture enforces its own timeout (D4) and releases the lock on exit, so the record is read once the lock is free.
- capture `running` with its lock free, or `failed` — Stuck naming the log.
- capture `passed` — Done when HEAD is still the capture's commit, writing `awaited_at` into the durable scope record and committing it with the capture log; Stuck when HEAD moved.

A Stuck while HEAD is still the branch point names `lyx drift capture` followed by `lyx loom resume`, which captures again on the branch point and re-enters `Drift-Await` (D4), or `lyx drift defer` followed by `lyx loom resume` to go on with the regression question deferred.
A Stuck after HEAD moved names only `lyx drift defer` followed by `lyx loom resume`, since no capture may run on changed code.
`lyx drift defer` writes `awaited_at` into the scope record, marked as a deferral, and refuses unless the run is halted at `Drift-Await`.
After a deferral past a failed or missing capture, `Drift-Verify` halts a second time, naming `lyx loom goto Describe` (D6); after a deferral past a HEAD-moved halt the passed capture still exists, so `Drift-Verify` compares against it.
`awaited_at` in the durable scope record lets a later entry, after Webster's reroute through the plan perch, pass without a baseline check, a deferred one included.

An adopt on another machine carries the scope record but not `.lyx/drift/`, which is never tracked, and the run's processes on the old machine are ended, a running capture included.
So the adopting machine has no capture record or baseline.
In loom, before `awaited_at` is set, `Drift-Await` halts on the missing capture record, and on the branch point `lyx drift capture` with `lyx loom resume` captures there; once `awaited_at` is set, `Drift-Await` passes and `Drift-Verify` halts on the missing capture, naming `lyx loom goto Describe`, since no capture may run on changed code.
In darn, an adopt while `Drift-Baseline` is the current row re-enters it and captures again (D3); a later adopt leaves `Drift-Verify` to halt on the missing capture, naming a goto to `Code-Bouncer` (D6).

Rationale: no code may change before the baseline exists, and a failed capture never falls back to one taken later.

### D6. `Drift-Verify`: compare, then one strong agent on drift

The row reads the scope record: no record or scope `off` returns Done with that reason.
A scope with no passed capture record halts Stuck, naming `lyx loom goto Describe` (darn: `Code-Bouncer`) as the way to go on without regression.
Otherwise it runs the compare mechanically, holding one gate slot and ended at `compare.timeout_min` as D4 ends a capture, with `{skip_file}` listing the scope's skipped tests and the accepted ones (D7).
A compare passes on exit code 0.
A passing compare returns Done and spawns nothing.

A failing compare spawns one agent on `drift.verify_model`, given the compare's log, the scope record, `baseline.md`, the decision record as amended (loom) or board entry (darn), and the baseline's commit.
Within its budget it judges which drift is acceptable against the task and `baseline.md`'s expectation, accepts that drift (D7), finds the cause of the rest and fixes it, committing each fix, until the compare passes.
An amendment after Discussion never re-scopes; the agent judges drift against the amended record.
It runs regression only through `lyx drift compare` and `lyx drift capture --side`, whose runs count against `drift.verify_max_runs` in a Go-written counter; past the budget both verbs refuse, naming the report and the stop as the way forward.
Both verbs refuse unless the run's current row is `Drift-Verify`, naming that row as the place they run.
The row resets the counter at each fresh spawn of its agent, so each attempt has `verify_max_runs` runs and a further attempt takes a resume by the parent; the row's own mechanical compares never count.
A side capture writes to a fresh numbered directory under `.lyx/drift/side/`, renders `{baseline_dir}` as that directory, needs no clean tree or branch point, and writes neither the capture record nor the baseline directory.
The agent writes its reasoning, every acceptance and every fix to `_lyx/drift/verify-report.md`, the row's output.
It stops without passing the compare when it is unsure or past its budget.

When the agent ends, the row's handoff runs in this order on every attempt: a dirty tree re-prompts the session to commit; then the `verify` gate over the hub's new `drift-verify` gate key (scope `diff`) through `verifytree.Verify`, when the session committed code; then the compare, last, so no fix the agent or a gate re-prompt makes goes uncompared.
A dirty tree or a failing test gate re-prompts the session up to three times, then is Stuck.
A failing final compare is Stuck at once, naming the report and the ways forward: `lyx drift accept` and resume, or fix and resume.
Under scope `full`, an attempt in which the agent session accepted drift (D7) is Stuck even on a passing compare, so the parent rules on drift in a task meant to change nothing.
That halt names the parent's ways forward: `lyx loom resume` to stand by the session's acceptances, after any `lyx drift accept` of the parent's own, or `lyx drift withdraw` and resume to reject one; the resumed attempt, its compare passing with no acceptance by its own session, returns Done.
Under scope `full` every halt over drift or a missing baseline, this one, a failing final compare and the halt on no passed capture record, has a reason opening with `shedadapters.DecisionHaltPrefix`, so `arbiterstop.HandsBack` marks it and the stop chain hands it to the parent ahead of the tackler and the arbiter.
The row commits the report, the scope record and its logs with the run's records on every way out.
No review follows the agent's fixes, and the agent is no perch.

Rationale: small unwanted drift is beyond a Sonnet implementer and takes a strong agent with tracing and many varied runs; one agent both judging and fixing is the operator's design.

### D7. Accepting drift without touching the baseline

`lyx drift accept <test> --reason <text>` appends the test and reason to the scope record's accepted list; every later compare skips it.
Each accepted entry carries who accepted it: the row's live agent session, with the attempt it ran in, or a caller at a `Drift-Verify` halt.
`lyx drift withdraw <test>` moves an accepted entry to the scope record's withdrawn list, so the next compare runs the test again.
The agent session's `lyx drift accept` refuses a withdrawn test, naming the parent's ruling, so a rejection holds across resumes; only a caller at a halt accepts it again.
Both verbs refuse unless the run's current row is `Drift-Verify`, which a Drift-Verify halt keeps as the current row, so only that agent, the arbiter under scope `partial`, or the operator at that halt can accept or withdraw.

Under scope `partial` an acceptance can let through up to every test the compare runs, a compare left with no test included, and the row then returns Done with no review.
The agent's judgement, against the task and `baseline.md`'s expected drift, is the only bound on what it accepts, and the arbiter may accept under `partial` at a halt that names `lyx drift accept`; every acceptance and its reason stands in `verify-report.md` and the committed scope record.
Under scope `full` the agent's acceptances end the attempt in a decision halt (D6), so the parent bounds them.

Drift the agent traces to a parent commit merged after the baseline's commit is acceptable, and the report names the commit.
Nothing in this design deletes, overwrites or refreshes a baseline after the first code change.

Rationale: an accepted drift must let the compare pass without blessing it in the baseline, and the test is the granularity both the harness and the skip file speak.

### Added after Discussion (parent, 2026-10-25): Drift-Verify sits after the Code perch, before Describe

Decision: In loom, Drift-Verify runs after Code-Bouncer and before Describe, so it compares the code as it lands: the task's changes, the Code perch's fixes and the parent commits Parent-Align merged in. Drift caused by merged parent commits is judged under D7's parent-commit acceptance. Answers review finding F9.

Rationale: Operator ruling, 2026-10-25. Since fc75aef79 Parent-Align runs before the Code perch, so the entry's placement before Parent-Align is no longer possible.

### Added after Discussion (parent, 2026-10-25): Drift-Await is skipped through its entry gate when regression is off

Decision: Drift-Await gets entry_gate: drift.enabled like the other Drift rows, and with the flag off it records Skipped and runs nothing. Every producer's entry gate is the boolean check for whether it runs at all; no special case keeps Drift-Await ungated. Answers review finding F10.

Rationale: Operator ruling, 2026-10-25: with the flag off the Drift rows skip, and the entry gate is the mechanism for that.

## Constraints

- The design departs from no PATTERN entry.
  `_lyx/drift/` holds the scope record, the two reports and the logs; `.lyx/drift/` holds the capture record and lock, the baseline and side directories, the skip file and the run counter; only `internal/lyxdirs` names them.
  Every refusal and halt names its way forward and a test reaches each; `lyx drift` verbs stay allowed to agents.
  Both stencils are read through `internal/stencilstore` and render the parent and edit directives; both models go through `NewModelChoice`.
- The capture runs in the task worktree, never in a separate checkout.
- Between `Drift-Baseline` and `Drift-Await` no row builds in the task worktree: Discussion and Plan rows write only records, and the plan gate's type load is Go-only.
- Models (issue #609, 2026-10): the project root is `src/csharp/NORCE.Models/`, not the git root, hence `dir`; compare mode is `-e NORCE_BASELINE_MODE=verify`, where a missing baseline fails; `NORCE_BASELINE_DIR` sets the capture directory.
- `pairteardown` ends a pair's processes, the capture included, through `runproc.EndSparing`.

## Auto-mode assumptions

- The default models and budget in D1, the skip file in D1, the log location in D4, D6's own `drift-verify` test gate over the agent's fixes, D5's `lyx drift defer` and D7's `lyx drift withdraw` are auto-picks.

## Open risks

- The board entry places `Drift-Verify` "after Webster and the Code perch, before Describe and Parent-Align", but since commit `fc75aef79` (2026-10-24) `Parent-Align` runs before the Code perch, so both cannot hold.
  This record keeps "after the Code perch, before Describe" and lets the agent accept drift traced to merged parent commits (D2, D7); the parent may prefer `Drift-Verify` between Webster and `Parent-Align`, which leaves the Code perch's fixes uncompared.
- Models-side facts this machine cannot check, for OSL-1046: whether the harness reads `NORCE_BASELINE_DIR` in verify mode, whether its all-`Inconclusive` capture exits 0 under the `dotnet test` runner it uses, and whether its compare fails when it discovers no test; the harness also reads no skip file yet.
  D4's pass rule asks for written files as well as exit 0, so a capture that ran nothing fails either way.

## Acceptance criteria

- With `drift.enabled` false, a loom and a darn run record `Drift-Baseline` and `Drift-Verify` as `Skipped`, a loom run records `Drift-Await` as Done, and neither spawns an agent or runs a command.
- With the flag on, an undeclared `capture` or `compare` is refused naming `lyx config drift`, and a command without `{baseline_dir}` is refused at load.
- Loom: the capture starts detached and `Drift-Baseline` returns before it finishes; `Drift-Await` waits on a running capture while its lock is held, with no bound of its own, a capture waiting for a slot included, and passes when it ends passed.
- Darn: `Drift-Baseline` returns only after the capture passed, and is Stuck on a failed one; re-entered after its process died mid-capture, it captures again.
- Scope `off` captures and compares nothing; `partial` writes the skip file and passes it to capture and compare; `lyx drift scope` refuses the invalid combinations and outside `Drift-Baseline`.
- A failed capture, a timed-out capture whose tree was killed and whose record says `failed`, a capture whose lock is free while its record says `running`, a missing capture record under a non-off scope, and HEAD moved before the first await each halt `Drift-Await`, naming the way forward D5 gives for the case.
- At a `Drift-Await` halt on the branch point, `lyx drift capture` after a failed or dead capture captures again and `lyx loom resume` then passes `Drift-Await` without re-running `Plan-Write`; a main capture refuses off the branch point, outside `Drift-Baseline` and a halted `Drift-Await`, and on a dirty tree.
- `lyx drift defer` refuses unless the run is halted at `Drift-Await`; after it and `lyx loom resume` past a failed or missing capture, `Drift-Await` returns Done on every later entry and `Drift-Verify` halts naming `lyx loom goto Describe`.
- A capture started by `lyx drift capture` at a `Drift-Await` halt keeps running after the caller's process exits.
- After an adopt on another machine in loom, with `awaited_at` unset `Drift-Await` halts on the missing capture record, and with it set `Drift-Verify` halts naming `lyx loom goto Describe`; in darn, `Drift-Verify` halts naming `Code-Bouncer`.
- A capture exiting 0 with no file written fails; an existing baseline directory is moved to a backup, never overwritten.
- After `awaited_at` is set, `Drift-Await` re-entered after Webster committed code returns Done.
- Capture and compare each hold a gate slot and wait while every slot is taken.
- A passing compare in `Drift-Verify` spawns no agent; a failing one spawns one, and the row is Stuck when the final compare still fails.
- A test fix after a gate re-prompt is compared again before Done, and a dirty tree after the session is re-prompted.
- Under scope `full`, an acceptance the agent session made ends `Drift-Verify` Stuck in a decision halt the arbiter never settles, and the parent's accept and resume then reaches Done on a passing compare.
- `lyx drift compare` and `lyx drift capture --side` refuse past `verify_max_runs` and outside `Drift-Verify`, and a fresh agent spawn resets their counter; a side capture leaves the baseline directory and capture record untouched; `lyx drift accept` and `lyx drift withdraw` refuse outside `Drift-Verify`, an accepted test is skipped by the next compare, and a withdrawn one is run again and refused to the agent session's `lyx drift accept`.
- The scope record, reports and logs are committed with the run's records.
- Tests use fake capture and compare commands; none needs Models.

## Notes for the plan writer

- The new rows need `Name*` constants beside loom's and darn's existing ones for the coverage guard.
- `shedadapters.DecisionHaltPrefix`'s doc names only the bouncer's decision halt; D6's scope-`full` halts widen its meaning, so its doc, or a shared hand-back marker it moves to, covers them.
- `Env.SettingKnown`, which an entry gate on a `module.key` operand needs, is filled in `internal/loomcli/wiring.go`; darn's recipe build must fill it too.
- The C# sandbox hub at `/home/knatte/Code/lyx-csharp-LYXHUB` is where board note `csharp-sandbox-hub` proves the Drift rows end to end with its own harness, outside this task; the Models run on OSL-1046 is the final confirmation.
