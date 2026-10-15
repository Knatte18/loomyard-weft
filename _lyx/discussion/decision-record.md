# Decision record — shed-tackler

## Goal

Every shed-driven run can be driven by the Go driver with the tackler behind it: loom, darn and batten today, and any later recipe by supplying seams.
The Go driver and the tackler move out of `internal/loomcli` into a shed-layer package, shed gets its own config module holding the defaults, and a module's own config overrides them.
The first live run of the Go driver happens on this shed version, not on today's loom version.

## Scope

In:

- A new shed-layer package holding the Go driver loop, its rule table, settle records, stop reports, the hand-over, the pause hold, and the whole tackler (attempts, stop key, worktree snapshot, spawn through shuttle, verdict validation, consume, tamper check, prompt fill).
- Loom and darn runs driven by it through `lyx loom drive` when seeded `go`; batten's `lyx batten run` driving its own run through the same body in-process.
- A new `shed` config module (`shed.yaml`) with `driver_kind`, `tackler`, `tackler_timeout_min`, `tackler_max_spawns`, and per-module overrides in `loom.yaml`, `darn.yaml` and `batten.yaml`.
- The tackler stencil moved to a shed stencil, made recipe-neutral, with a recipe-description marker, a list of stop kinds with their usual verdicts, and a reading order.
- Docs, PATTERN entries and refusal-spec rows that follow the move.

Out:

- The arbiter stays loom's code and config (`lyx loom arbiter`, `loomengine` arbiter settings); the driver reaches it only through a seam.
- The LLM driver, the `lyx loom` run-verb namespace (`run-verbs-shared`), and the hub-level run driver (`orch-split`).
- A message receiver for go-driven runs (`orch-split`'s routing).
- Turning `driver_kind` to `go` anywhere: every default stays `llm` for loom and darn.

## Decisions

### D1 — One shed-layer package owns the Go driver and the tackler

Decision: a new package `internal/sheddriver` holds the driver loop, rule table, settle records, stop reports, hand-over, pause hold and the tackler, moved from `internal/loomcli` and `internal/loomengine`.
The types both the driver and loom's arbiter use, the stop key, the worktree snapshot and the settle record, move with it, and loom's arbiter imports them from `sheddriver`.
It is a told-geometry engine: it imports no `lyxcwd`, no `loomengine`, no `loomshed` and no `<module>cli`, takes every path told, and gets recipe-specific behaviour through a seam struct the arming module fills.
It joins `PATTERN-told-geometry`'s bound packages, and a seam enforcement test pins its imports.
Rationale: the rule table and the tackler stencil are recipe-blind in what they decide; what binds them to loom is wiring and a few loom-declared types, so the code moves and the wiring becomes seams, the split `shedverbs` uses.

### D2 — The seams a recipe supplies

Decision: the arming module fills these seams; every optional seam left unwired ends in a hand-back or a skipped check, as stated.

- Step and goto: `shedverbs.RunUntilStop` and `RunGoto` over the run's own `Spec`.
- Interrupt policy per row (`reinvoke` or `handback`): loom and darn tell `loomshed`'s per-row map as today; batten tells `reinvoke` for Run-Shed and `handback` for every other row. An unnamed row reads `handback`, as today's empty policy does.
- Arbiter: optional. Loom and darn wire `lyx loom arbiter`'s in-process path as today. Batten wires none; with no arbiter, a stop the rule table sends to the arbiter hands back, and a tackler `escalate` hands back with its case brief named in the hand-back notice.
- Hand-back: writes the stop report, commits the run's records where the recipe commits them, writes the park marker, and queues the parent notices. Loom and darn keep today's notify rules.
- Verify-failure goto target: loom and darn tell their Publish failure record target as today; batten tells none, so that row never fires for batten.
- Watched read: loom and darn read their watched marker; batten is never watched.
- Friction note writer: loom and darn write the `go-driver-paused` note; batten writes none.
- Hand-over spawn: optional; loom and darn spawn `lyx loom drive` of the on-disk build. Batten wires none, and with none the hand-over row never fires.
- Tamper snapshot: optional; loom and darn snapshot the task worktree as today. Batten wires none (D4).
- Resume goto targets: loom and darn allow the current row and the routed-from row as today; batten allows the current row only, and a verdict naming any other row is refused and hands back.
- Recipe description: one short line filled into the tackler stencil (D7).

Rationale: these are the loom-specific parts of today's `drivewiring.go` and `driverules.go`.
Bound: an unwired seam removes a row or an agent from the table; a stop whose row never fires meets the later rows unchanged, so it ends in the tackler under its spawn budget and stop-key check, a re-step under the repair cap, or a hand-back, and no unwired seam adds a resume path the table does not already have.

### D3 — `lyx loom drive` stays the verb for loom and darn

Decision: the hidden role verb keeps its name and mounts `sheddriver`'s body with loom's seams; it serves darn runs as the other `lyx loom` run verbs do.
No `lyx shed` verb is added.
The records `shedrun` already declares per run-id keep their paths unchanged (`PATTERN-shed-run-directory`): the drive reports (`DriveReportsDir`), which the LLM driver writes too, the driver lock, the park marker, the resume report, and the tackler directory, attempt directories and marker.
The settle records, the Go driver's only record that changes home, move from loom's friction directory (`SettleRecordsFile` over `LoomFrictionDir`) to a per-run `shedrun` path, and every reader follows: the driver, `internal/arbiterstop`'s hold reading and batten's publish retry (D8).
The driver log stays a told path; loom tells `LoomDriverLog` as today, since `lyx loom start`'s handshake names it.
Rationale: `run-verbs-shared` keeps run verbs under `lyx loom` for now and keeps `lyx shed` to the FSM; the driver queues parent notices and parks runs, which is that layer.
No run holds settle records at the old location today: no loom or darn run is seeded `go`, and batten runs, though seeded `go`, run no Go driver, so the settle records move without a migration.

### D4 — Batten drives its own run through the driver body

Decision: `lyx batten run <slug>` runs the `sheddriver` body in-process over its own run instead of `shedverbs`' generic `run` body, so an unexpected batten stop meets the rule table and the tackler before the orch.
`PATTERN-shed-verb-set` gains a clause: a recipe's `run` verb may be `sheddriver`'s body, which steps through `shedverbs`' `--until-stop` and goto bodies and restates neither.
Batten arms the `--until-stop` loop (`Spec.Loop`) without the agent idle watchdog, since Run-Shed's wait is idle by design, and without an arbiter attempt hook.
The process takes the driver lock for its life; a second `lyx batten run` for the slug is refused by the driver lock.
A rerun after the process died while a detached step still runs adopts that step through the loop's waiter instead of exiting as busy.
A batten run started on the old build is not adopted: the old process holds the run lock and writes no loop pid file, so a new-build `lyx batten run` started while it is alive meets that run lock as a busy stop and exits, leaving the old process to finish on its own build, and a rerun after the old process has exited resumes at the recorded row.
Batten's tackler strand runs in the prime's reed session through a detached runner from batten's CLI wiring, so Worktree-Teardown's session shutdown never ends it.
Batten wires no tamper snapshot, since the prime has other writers (landings, the orch) and a snapshot would read `tampered` falsely.
With no snapshot, a batten tackler that writes in the prime (files, the index, a local commit) has its verdict acted on, and a resume may then re-enter Land or Worktree-Teardown over that state.
The only bound on a batten tackler's writes is the stencil's write rule, which allows the verdict and the case brief only, and it is prompt text that nothing enforces: no shuttle deny stops Edit, Write, `git add` or `git commit`, and the stop-key check bounds run movement, not the tackler's writes.
No mechanical write check is added, since the prime's other writers leave no baseline a check could compare against; the residual risk is recorded under Open risks.
A tackler `resume` for batten names only the halted row, through D2's resume-goto seam, so it does what a rerun of `lyx batten run` does today: re-enter the row it halted at, Land and Worktree-Teardown included.
Batten's hand-back queues one orch notice per stop episode through `orchengine.QueueNotice`, sharing the rows' per-episode marker keyed by batten's history length, so a row notice already sent is not repeated; a paused stop sends none, since a pause is the operator's own act.
Batten's window, pause, row notices, `lyx batten run`'s envelope, `abandonedSession` and `haltedOverDoneChild` flag keep their behaviour.
Rationale: batten already steps in Go; a private stop handler beside the driver would be the second loop the operator's one-pattern goal rules out.
Bound: the tackler can only re-enter batten's halted row, never move the child run, and never resume without passing the stop-key check; what it writes in the prime is bounded only by the stencil's write rule, which is not enforced.

### D5 — The `shed` config module and the overrides

Decision: a new config module `shed`, `shed.yaml`, declared by `internal/sheddriver`, registered in `configreg`, not hub-wide, its keys open to the local layer so a task's `run_settings` can set them.
Its keys and defaults are today's: `driver_kind: llm`, `tackler: haiku`, `tackler_timeout_min: 10`, `tackler_max_spawns: 4` (0 turns the tackler off).
`loom.yaml` keeps its four keys as overrides and `darn.yaml` gains the four, both open to the local layer; `batten.yaml` gains the three tackler keys and no `driver_kind`, since batten's own driver is always `go`.
Every new key is classified in its module's `ConfigSpec`, and each module's `CheckValues` validates its override values as loom's does today.
An override that is empty or absent inherits the `shed.yaml` value; a set override wins, and `tackler_max_spawns: 0` set in a module still turns the tackler off for that module.
A `tackler_timeout_min` of 0 in a module inherits the next layer, as an unset one does, and a 0 in `shed.yaml` takes the built-in 10 minutes.
The plan picks the representation that tells an unset `tackler_max_spawns` from 0.
Resolution is one pure function in `internal/sheddriver` taking the shed config and the module's override; only `configengine` reads the files (`PATTERN-config-layers`).
The arming module passes the override of the run's own recipe for every tackler key: `loom.yaml` for a loom run, `darn.yaml` for a darn run, `batten.yaml` for batten's own run; `lyx loom drive`'s wiring picks it by the seed's recipe, the rule D6 states for `driver_kind`.
`shed` loads through `LoadOrTemplate` (`PATTERN-config-strictness`), so a hub or pair with no `shed.yaml` yet reads the template defaults.
Loom's `Migrate` hook runs on every reconcile and on every task's `run_settings` document (`internal/configreg/runsettings.go` calls `module.Migrate`), so a rewrite keyed on value equality alone would fire forever, erase a deliberate pin, and in `run_settings` drop a task's pin back to the hub's value.
The migration is therefore one-time and hub-level, in two steps of one pass.
The value rewrite fires on a `loom.yaml` only when all four keys hold the old template defaults (`llm`, `haiku`, 10, 4) together with the old template's comment on `driver_kind`, and turns the four into the inherit value.
The comment refresh runs after it: each of the four keys whose comment equals the old template's text for that key gets the new template's comment, with its value untouched, since reconcile never refreshes an existing key's comment and the old comments say empty takes haiku and 0 takes 10.
A hub that pinned any of the four keys so gets the new comments without the value rewrite, and a key whose comment was edited by hand keeps it.
Both steps are idempotent: the old comment text is gone after the pass, so neither fires again.
A `run_settings` document carries no template comment, so the migration never touches it, and a value set anywhere after the migration, `llm`, `haiku`, 10 and 4 included, is kept.
The loom template's four keys, `darn.yaml`'s four and `batten.yaml`'s three ship with the inherit value, so a freshly seeded hub does not mask `shed.yaml`.
Key names stay today's loom names; `loom-model-key-names` renames them later in every module at once.
Rationale: shed holds the defaults and a module overrides them, as the board asks; without the migration, the hub's `loom.yaml`, which spells every default out, would make `shed.yaml` dead for loom runs.

### D6 — Each recipe resolves its own `driver_kind`

Decision: a run's seeded driver resolves from its own recipe's module override, then `shed.yaml`: loom runs from `loom.yaml`, darn runs from `darn.yaml`.
This holds at both seeding sites, `loomcli`'s shared bootstrap and batten's child-driver reader; both pick the module by the child's recipe.
The typed `--child-driver` precedence and the rule that a recorded driver never changes stay.
`PATTERN-driver-choice-single-site`'s background and its tripwire in `internal/loomcli/bootstrap_test.go` follow: batten's child-driver carve-out is re-keyed to its new shape, and `sheddriver` reads no seed driver field.
Rationale: today a darn run reads `loom.yaml`'s `driver_kind` at both sites; resolving per recipe gives darn its own key without loom's choice leaking into it.

### D7 — One recipe-neutral tackler stencil, sharpened

Decision: the stencil moves to `contracts/stencils/shed/shed-template-tackler.md`, read at call time through `stencilstore`.
Its loom-specific text goes: the name says shed driver, the log marker is the shed driver log, and `escalate` says a stronger agent or the parent takes the case, without naming the arbiter.
It gains:

- a required `{{.recipe}}` marker: the recipe name and the one-line description the arming module tells;
- a short list of the stops it receives (a transient error, `producer`, `bootstrap`, `unseeded`, `interrupted`, `blocked`, a returned stop it already resumed), each with how it typically looks and its usual verdict;
- a reading order: the envelope and the stop report first, then the stop step's trace copy and stderr, and the child traces only when those leave the cause open.

Its goto rule names the rows D2's resume-goto seam allows for the run, so a batten tackler is told the halted row only.

Rationale: the board's stencil points, for a small model on its first live runs.
The marker is required rather than optional because every arming module supplies the line, so a missing value is a wiring fault.

### D8 — Batten's child reads follow the records' new home

Decision: batten's publish-retry read of the child's settle records goes to their per-run `shedrun` path of D3; the InnerRun hold's tackler marker and driver lock reads and the halted-child re-step keep the `shedrun` paths they read today.
Their semantics do not change.
Rationale: the settle records are the one child record that moves.

### D9 — New PATTERN entry for the driver chokepoint

Decision: add `PATTERN-shed-driver` under Shed and loom, with a background file: "Driving a shed run in Go, deciding its stops or spawning a tackler: only `internal/sheddriver`, told its paths and seams." (test)
The seam enforcement test pins its imports; that no `<module>cli` or producer runs a step-until-stop loop of its own is review discipline, as `PATTERN-shed-verb-set`'s first clause is.
Batten's InnerRun wait, which polls a child it does not step, is outside it.
Rationale: without the entry, a later recipe would grow a private loop as batten has today.

## Constraints

- These entries bind the moved code as they bound it in `loomcli`: `PATTERN-shed-run-directory`, `PATTERN-config-layers`, `PATTERN-config-strictness`, `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-spawn-color`, `PATTERN-agent-name`, `PATTERN-shuttle-stop`, `PATTERN-spawn-observability`, `PATTERN-daemon-wakeups`, `PATTERN-tolerant-records`, `PATTERN-refusal-way-forward`.
- `PATTERN-told-geometry`, `PATTERN-shed-verb-set` and `PATTERN-driver-choice-single-site` change as D1, D4 and D6 say.
- `PATTERN-orch-pane-single-writer`: batten's hand-back notice is queued through `orchengine`, never typed.
- `PATTERN-sandbox-coverage`: the `shed` config module is covered or excluded with a reason.
- The rule table's rows and order, the repair cap of two re-steps per stop key, the verdict vocabulary and the attempt records move unchanged; only their inputs become seams.
- The LLM driver keeps working for loom and darn.

## Auto-mode assumptions

- The rule table is fit for batten's stops as it is, with only the interrupt policy told; tuning it for batten is not asked.
- The new stencil content is written from the rule table and today's stencil, with no transcript of the tackler's live behaviour, since it has never run live.

## Open risks

- A batten tackler that writes files, stages changes or commits in the prime, against the stencil's unenforced write rule, leaves that state in place, and a later Land, reached by its resume or by the run's own progress, can carry it into the push and the deploy that follows.
- A killed batten step child is now an `interrupted` stop; Run-Shed re-steps under the repair cap, and a child killed a third time at the same stop goes to the tackler.
- Batten's notices: the hand-back notice and the rows' notices must share episode bookkeeping, or the orch gets a stop twice.
- A go-driven loom or darn run has no message receiver, as today's go-driven loom run has none; `orch-split` owns that.
- `twb-d0-sa` plans a darn decision record and PR-Align, which may change what darn's hand-back points at; D2's darn seams follow loom's today.
- The tackler has never run live; its first live runs are on this version, so stencil and rule-table faults surface then.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - migration-comments-stale-on-pinned-key-hubs (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `lyx loom drive` drives a go-seeded loom run and a go-seeded darn run through `sheddriver`, with the rule table, tackler and hand-back behaving as today's loom tests pin.
- A loom or darn run seeds `driver_kind` from its own module's override, then `shed.yaml`, then `llm`; a typed `--child-driver` still wins; a recorded driver never changes.
- With no `shed.yaml` present, seeding and driving read the template defaults.
- Empty or absent module overrides inherit `shed.yaml`; a module's `tackler_max_spawns: 0` turns its tackler off while `shed.yaml` says 4; a module's `tackler_timeout_min: 0` inherits `shed.yaml`'s value, and a `shed.yaml` 0 takes 10 minutes; an invalid override value is refused, and `driver_kind` in `batten.yaml` is refused.
- A darn run driven by `lyx loom drive` with `darn.yaml`'s `tackler_max_spawns: 0` and `loom.yaml`'s 4 spawns no tackler, and with the two values swapped it spawns one.
- Migrating a hub `loom.yaml` holding the old template defaults and comment leaves the four keys inheriting, each carrying the new template's comment, and a second reconcile changes nothing; a `loom.yaml` with one key off the old defaults keeps that key and every other value, each of the four keys carries the new template's comment, and a second reconcile changes nothing; a `run_settings` document setting `tackler_max_spawns: 4` over a hub value of 0 keeps its 4.
- A freshly seeded hub's `loom.yaml`, `darn.yaml` and `batten.yaml` hold the inherit value for every override key.
- `lyx batten run` over a blocked or error stop spawns a tackler in the prime's session; a `resume` re-enters the halted row, and a `resume` whose goto names the routed-from row is refused and hands back; a `hand-back` queues one orch notice per stop episode, not repeated by a row notice for the same episode; an `escalate` hands back naming the case brief; a requested pause sends no notice.
- A batten stop after the on-disk build changed goes to the tackler, never a hand-over; a batten blocked stop with a Publish failure record present goes to the tackler, never a verify goto.
- The rendered batten tackler prompt allows writes to the verdict and case-brief paths only and names the halted row as the only goto target.
- The rendered tackler prompt for loom, darn and batten names no arbiter and contains D7's stop-kind list and reading order.
- An interrupted Run-Shed step re-steps; an interrupted step at any other batten row hands back.
- A second `lyx batten run` for a running slug is refused while the first holds the driver lock; a rerun after the first died adopts a still-running detached step; a new-build `lyx batten run` started while an old-build batten process holds the run lock exits as busy, and a rerun after that process exited resumes at the recorded row.
- Batten's InnerRun hold sees a child's live tackler marker at its `shedrun` path, and batten's publish retry reads the child's settle records at their new per-run path.
- The tackler prompt renders with the recipe marker filled for loom, darn and batten, and a missing marker value is refused.
- Scenarios kept from today: a driver restarted over a halted run reuses an unclosed tackler attempt; a pause covering the slug holds spawns and re-steps; a loom tackler that changes the worktree reads as `tampered`; a run moved by the orch during a tackler is decided afresh; a tackler timeout hands back with cause `timeout`; tackler attempts count across a driver restart of the same run.
- `internal/sheddriver` imports no `lyxcwd`, `loomengine`, `loomshed` or `<module>cli`, pinned by its seam test.

## Notes for the plan writer

- Today's code: `internal/loomcli/drive*.go`, `driver{rules,report,resume,resumeline,spec}.go`, `tackler*.go`, `drivewiring.go`; `internal/loomengine/tackler.go`, `settle.go`, the arbiter stop key and `driver.go`'s `ResolveDriverKind`; `internal/loomshed/interruptpolicy.go`; seeding at `internal/loomcli/sharedbootstrap.go` and `internal/battencli/wire.go`'s `ChildDriver`; batten's child reads in `internal/battenshed` and `wire.go`'s `GoVerifyRetrySpent`; loop arming in `internal/loomcli/looparm.go`.
- `driverprompt.go`, `driverlaunch.go` and the driver strand belong to the LLM driver and stay in `loomcli`.
- The tackler spawn takes its engine from the same place the moved `loomcli` code does, so `llm-agnostic-engine` gains no new site.
- Check `cmd/lyx/testdata/test-budget.yaml` for the new package and for `loomcli`'s row as tests move.
