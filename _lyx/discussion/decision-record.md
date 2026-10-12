# Decision record: Go driver per task, with the tackler and the arbiter behind it

## Goal

Make the Go driver (`shedrun.DriverGo`) a full alternative to the LLM loom driver for a task's own loom run.
Go steps the run and settles the mechanical stops by a closed rule table.
A cold, cheap agent, the **tackler**, is spawned only at a stop no rule settles.
The arbiter is spawned only when Go's own rule or the tackler's verdict calls for it.
Anything needing a parent choice still goes to the orch.

## Scope

In:

- The per-task driver choice through `run_settings`, with the LLM driver as the default.
- The Go driver: a settle loop over the `--until-stop` step loop, with its own liveness signal, spawned by `lyx loom start` for a go-seeded run.
- The tackler: its stencil, its spawn through shuttle, its verdict and case brief, its attempt records, and its loom config keys (model-spec, timeout, spawn budget).
- The arbiter receiving the tackler's case brief, and the Go driver calling the arbiter's existing spawn path in-process.
- `lyx loom resume` spawning a Go driver for a halted go-driven run that has no live driver.
- Batten: re-stepping a halted, driverless go child after a binary change; reading the Go driver's liveness; holding its stop notice while a tackler is in flight.
- One line in the orch's role stencil saying a go-driven run has no driver to message.
- The empty-reason `paused` stop: shed sets a step-result reason on each `paused` stop that returns none, and the Go driver reports any that still meets it through a Go-authored friction note.

Out:

- Any change to the LLM driver, its stencil or its fork-and-guide repair path.
- Repair verbs for the tackler or the Go driver: neither deletes branches, edits files or fixes git.
- Narrowing the arbiter's eligibility list (`arbiter-when`) and the soft and hard review caps (`judge-cap-call`).
- The fix of `arbiter-relaunch-moved`.
- Batten's own driver, which is already Go, and the hub-level Go loop of `orch-split`.

## Decisions

### D1. The per-task choice is a loom config key, set per task through `run_settings`

Decision: a new loom config key `driver_kind` takes `llm` or `go`, defaults to `llm`, and is open to the local layer, so `run_settings` sets it per task (Seed-Child writes a task's run settings as the worktree's local layer before it seeds the child).
Precedence at seeding: an operator-typed `--child-driver` on `lyx batten run`, then the resolved `driver_kind`, then `llm`.
The batten prime seed records `child_driver` only when the operator typed it, and an absent `child_driver` changes meaning from `go` to "resolve from `driver_kind`" in `childDriverOf` and the `ChildDriver` closure; `refuseAdoptedSeed` compares a typed flag against a recorded value, and refuses a typed `--child-driver` against a found seed that records none, naming the way forward (delete the seed to re-seed it with the flag, or drop the flag to take `driver_kind`), with its refusal-spec row; an untyped flag passes a seed that records none.
Seed-Child resolves `driver_kind` through `configengine`'s strict load of the task worktree's loom config, after writing the local layer; an invalid value is refused by `PrepareRunSettings` and routes Seed-Child to Stuck as other invalid run settings do.
Loom's own seed writer, which seeds a standalone `lyx loom start`, resolves an unseeded run's driver from the same key through the config start already loads strictly.
A seeded run's recorded driver never changes: editing `run_settings` after seeding has no effect on that run.
Rationale: run settings already reach a task as its local layer, so one key serves both a per-task choice and a hub-wide default, read only at the seeding sites `PATTERN-driver-choice-single-site` allows.

### D2. The Go driver is a hidden loom role verb with its own lock

Decision: a new hidden role verb, `lyx loom drive`, is the Go driver; the go arm of `lyx loom start` spawns it detached in place of `lyx loom run`, logging to the loom driver log.
`lyx loom run` stays the generic `shedverbs` run body, unchanged.
`drive` calls the `shedverbs` `--until-stop` step-loop body and the goto body, never restating them, and consumes the same stop envelope the LLM driver does, so each rule branches on the envelope's fields, its error `kind`, its `transient` class and `lyx shed status` facts, never on a row or recipe name or on reason text.
It has no reed strand and no pane.
For its whole life, settle waits included, it holds a driver lock of its own, declared in `shedrun` under the ephemeral tree; the run lock stays free between steps, so the orch, the operator and batten can still move a halted run while the Go driver waits on an agent (D12).
`drive` refuses at once when the driver lock is held, so a second Go driver never stacks.
Start's spawn decision, start's and resume's `driver_not_parked` refusal, batten's adoption, and batten's driver-dead notice each read a held driver lock as a live driver, beside the run lock and the reed strand they read today.
On start, `drive` removes the previous park marker and its resume-report file.
Rationale: the go arm already owns spawn and handshake for a go-seeded run; a lock of its own gives a settling Go driver the liveness the run lock gives only while a step runs, without blocking the actors that may move the run.

### D3. The Go driver's rule table is closed

Decision: at each stop the Go driver applies the first matching row, in this order:

| Stop | Action |
|---|---|
| `done` | Commit the run records and exit; a `friction: failed` done also notifies the parent (D8). |
| `busy` refusal | Write a settle record, log the lock holder, and exit. |
| `ownership` refusal, a refusal with no `kind` | Hand back. |
| `paused` | Hand back; an empty `reason` also writes a friction note (D13). |
| `awaiting` with a pending Bouncer escalation the arbiter's eligibility admits | Arbiter directly (D6), budget permitting; a `hand-on` or a refused spawn hands back, relaying any `parent_notice` (D8). |
| `awaiting` with a non-empty `parent_notice` | Relay the notice (D8) and hand back. |
| other `awaiting` | Hand back as a planned hand-off. |
| `blocked` with a verify failure record that matches the halt (D4) | Watched: hand back, leaving the halt to batten's publish retry. Unwatched: perform the goto and re-step once per run, then hand back on a later such halt (D4). |
| binary-eligible stop whose last step's recorded build differs from the on-disk `lyx` build (D5) | Hand over to a fresh Go driver of the on-disk build, which re-steps (D5). |
| the same stop key again after an applied tackler `resume` | Arbiter (D6), eligibility and budget permitting; otherwise hand back. |
| transient error | Tackler (D7), budget permitting; otherwise hand back. |
| `interrupted` under `reinvoke` | Re-step within the repair cap. |
| `interrupted` under `handback` or an empty policy | Hand back. |
| `producer`, `bootstrap` or `unseeded` error; a `reinvoke` stop past the repair cap; any other `blocked` stop | Tackler (D7), budget permitting; otherwise hand back. |

The repair cap is the guide's: at most two re-steps per stop key without the run advancing, the stop key being the current producer, history length and state.
A transient error reaching the Go driver has already been re-stepped once inside the loop, so, as the LLM driver does, the Go driver re-steps it no further.
A binary-eligible stop is the LLM driver's set: `blocked`, a `producer`, `bootstrap`, `unseeded` or `interrupted` error, or a transient that failed twice.
Rationale: each mechanical class of the 2026-10-12 count maps to one row; everything a row cannot name goes up one tier.

### D4. The Go driver's own goto is the verify failure record's, once per unwatched run

Decision: the only goto the Go driver performs by rule is to the target of a verify failure record that matches the halt, the record batten's publish retry reads; it never parses a target out of reason text.
A record matches under the same test batten's publish retry applies: a target and a write time present, the record's head equal to the child's HEAD, the halting history entry a Stuck of the record's row, and the write time after the previous history entry and at or before the halting one.
A record that fails the match drives no goto, and its `blocked` stop takes D3's last row.
When batten watches the run (the batten-watched marker names a live pid), the halt is batten's and the Go driver hands back.
The watched read is the existing pid-only read, so a stale marker whose pid was reused reads as watched; its bound is that the halt is parked with a stop report, and the Go driver sends D8's unwatched hand-back line at every verify halt it hands back as watched, so a false watched reading still reaches the parent.
When nothing watches, the Go driver performs the same one retry batten would: goto through the shed goto verb, which admits only a row at or before the current row, then re-step; a later such halt in the same run hands back.
The one retry per run is counted in two places, each written by its own actor and read by the other.
The Go driver records its retry as a settle record (D9) in the task worktree's friction directory, and batten's `publishRetryDecision` treats the run's one retry as spent when the child's settle records show a Go verify retry; batten already reaches that directory through the child's location.
Batten records its own retry in its publish-retry marker, which the Go driver reads only through a `battenshed`-exported read that takes the prime's scratch directory, and only when the batten-watched marker names a prime; with no prime named there is no batten retry to read.
So neither a Go driver restart nor batten adopting the run later retries again.
Halts whose way forward exists only as reason text (Finalize, say) go to the tackler, whose `resume` may name a goto within D7's bound.
Rationale: the verify failure record is the one structured goto target; batten's design already says a halt after its one retry needs judgement, and one actor per halt rules out the race of two movers.

### D5. A binary change re-steps through the on-disk build

Decision: the Go driver judges a binary change by the build identity of the `lyx` on disk against the identity the last step recorded, never by its own in-memory build; it reads the on-disk build identity itself and never uses the envelope's `binary_changed` field, which compares against the calling `lyx`'s build; any difference counts, a rollback included, since the on-disk build is the one the operator deployed.
A hand-over requires a clean tree: the task worktree has no uncommitted or untracked path outside the run's own records directories, read through the existing dirty-paths read; a dirty tree hands back with the paths in the stop report.
At a binary-eligible stop with a changed build and a clean tree, the Go driver hands over in this order: it writes the hand-over settle record and the stop report with the park marker, then releases its driver lock, then spawns a fresh `lyx loom drive` and exits; the fresh driver removes the marker on start (D2) and re-steps.
A crash or a failed spawn after the release leaves the run parked with a stop report: when watched, batten's InnerRun condition below re-steps it, and when unwatched, the Go driver sends D8's unwatched hand-back line before the release, naming the hand-over, so the parent learns of a hand-over that never came up.
One hand-over per stop key and build.
After a hand-back, the watch is batten's: its InnerRun wait re-steps a `blocked` or `failed` child through the child's bootstrap (`lyx loom start`) when reed holds no driver strand for it, neither the run lock nor the driver lock is held, the halt is not one batten's publish retry takes or is still taking, and the last step's recorded build differs from the on-disk build.
That is once per halt episode and build, under the same clean-tree check; batten records the re-step in a marker beside its publish-retry marker, naming the halt's stop key and the build, so a batten restart does not reset the cap.
An LLM-driven child always has a driver strand recorded, so the condition excludes it.
The check spawns a process, so its wait doubles from `notice_probe_s` while nothing changes, as `PATTERN-daemon-wakeups` requires.
Rationale: batten's InnerRun wait already polls every halted child for the whole run, and a Go driver that exits at hand-back never holds an old build across a deploy.

### D6. The arbiter is spawned in-process by the Go driver, through the existing spawn path

Decision: the Go driver calls the same function `lyx loom arbiter` runs, so eligibility, `arbiter_max_spawns`, attempt records, the `applied` mark, stop-key idempotence, deadlines and the verdict check are shared and unchanged.
It spawns the arbiter in three cases only: an `awaiting` stop with a pending Bouncer escalation the eligibility list admits; a tackler `escalate` verdict (D7); the same stop key returning after an applied tackler `resume`.
It checks eligibility and budget before each spawn; a refusal hands back.
A `resume` verdict is performed through the existing `--arbiter-attempt` re-step; a `hand-on` hands back with the arbiter's `question`.
Until `judge-cap-call` and `arbiter-when` narrow the list, the Go driver spawns the arbiter at every Bouncer escalation the LLM driver does today, soft-cap `budget` escalations included.
Rationale: one spawn path keeps one budget and one record set whatever the driver, and the narrowing those notes plan lands once for both drivers.

### D7. The tackler is a cold, single-stop, read-only agent

Decision: the tackler is spawned cold through shuttle, as an interactive strand in the pair's reed session like the arbiter, for one stop, and ends when its verdict is written.
Its stencil is a new lyx-owned stencil beside the arbiter's, recipe-blind, rendering the no-parent variant of the parent directive and the edit directive.
It reads the stop envelope, the Go driver's stop report so far, the step's trace copy, stderr and child traces, and the driver log.
It writes only `verdict.yaml` and, on `escalate`, the case brief, both in its attempt directory.
Its stencil tells it to run no `lyx` verb that changes the run and no mutating git command; that rule is stencil instruction only, as the arbiter's is.
Go bounds it after the tackler ends, against a record taken at spawn of the run's stop key, the worktree's HEAD, its checked-out branch, and its uncommitted paths with a content hash per path (the existing dirty-paths read plus a hash per path).
D12's status re-read runs first.
The run's records do not say which actor changed a stop key, since a goto or relaunch by the tackler and one by the orch or operator write the same history entry, so attribution is not decidable: a changed stop key is always a moved run, handled by D12 as a new stop, with the tackler's verdict discarded and a stop-report line noting the change.
With the stop key unchanged, a difference in HEAD, the branch, the uncommitted-path set or any path's hash reads as `hand-back` with cause `tampered`, since the actors that may move the run do so through verbs that change the stop key and leave the worktree alone.
Its verdict, alike the arbiter's in shape: `verdict` is `resume`, `hand-back` or `escalate`; `reason` always; `goto` optionally on `resume`, naming only the current row or the row the halting history entry routed from; `question` required on `hand-back`, addressed to the run's parent.
It escalates when it cannot name the cause from what it read, or when the cause needs code read and judged; a scope or design choice is a `hand-back`.
Go validates the verdict, and each of these reads as `hand-back` with a cause naming it: an unreadable or malformed verdict, an empty `question` on `hand-back`, an `escalate` with a missing or empty case brief, a `goto` naming any row other than the current row or the row the halting history entry routed from (cause `precondition`), a `resume` at a `blocked` stop without a `goto` (cause `precondition`, as the arbiter's), a spawn the provider never brought up ready, and a timeout.
Its deadline excludes intervals a pause covered the slug.
Attempts live under the run's tackler directory as `attempt-<N>`, written before the spawn and counted from disk, keyed by the stop key like the arbiter's: a re-run for a stop key already judged spawns nothing and returns that verdict, waits on a live tackler within its deadline, or stops one past it through shuttle's stop verb.
A tackler `resume` is performed through the arbiter's consume path: `shedengine.Relaunch` with an Admit that, under the status lock, refuses unless the stop key and pause conditions equal those the attempt recorded at spawn, and writes the `applied` mark in the attempt directory in the same hold.
A refusal is acted on as a new stop (D12).
The mark lets a restarted driver tell a `resume` never performed (perform it) from one performed whose stop key returned (arbiter row of D3).
A wrong `resume` or `goto` is bounded to one re-step, or one move to the current row or the row the halting history entry routed from, per attempt, so a cheap verdict never rewinds the run past the row that halted it and re-runs earlier LLM rows; attempts are capped at `tackler_max_spawns` per run, and a returning stop key goes to the arbiter row of D3.
While a tackler may be live, an ephemeral marker declared in `shedrun` names the attempt, its strand and its deadline; batten's notice hold reads it as it reads the arbiter's.
The status commit stages the tackler directory with the run records, and a pair's session end ends a live tackler strand with the pair's other strands.
New loom keys, all open to the local layer: `tackler` (model-spec, default `haiku`, empty takes `haiku`), `tackler_timeout_min` (default 10, 0 takes 10), `tackler_max_spawns` (default 4 per run, 0 turns the tackler off so every stop it would take hands back).
Rationale: a cold spawn per stop costs one process start, and the arbiter's record and idempotence scheme already survives restarts.

### D8. The Go driver notifies the parent by the LLM driver's two rules, through the orch notice queue

Decision: a go-driven run's parent is the hub's prime orch, and the Go driver queues each parent message through `orchcli.NotifyPrime`, the way loom's other Go notifiers reach it.
It picks the rule at each stop from the batten-watched marker:
watched, it sends only a `parent_notice` relay verbatim (after any arbiter outcome at that stop), a hand-back that carries a `question`, and a `friction: failed` done;
unwatched, it also sends one line naming the run-id and stop report at every other hand-back.
Every message ends with the tail "Answer briefly, then continue your task.", and every send or failed send is a line in the stop report.
With no orch strand recorded, the notice is logged and not queued, and the stop report alone carries it.
Rationale: the two rules already decide what the orch hears from an LLM-driven run; reusing them keeps the orch's view the same whichever driver runs.

### D9. Every hand-back leaves a Go-authored stop report, a park marker and settle records

Decision: at every hand-back the Go driver writes a stop report at a fresh drive-report path, commits the run records and writes the park marker through the same code `lyx loom commit-records --park` runs, then releases its driver lock and exits.
The report names the run, its position, the envelope's `reason`, every rule applied, every tackler and arbiter attempt with its verdict, the parent question when present, and the friction and repair directories.
Every Go action (re-step, goto, hand-over, spawn, hand-back, `busy` exit) appends one settle record into the friction directory, as the guide's repair records are: the stop key, the settler (`go-rule`, `tackler`, `arbiter` or `parent`), the outcome and its time.
The caps of D3 and D4 are counted from these records.
Rationale: batten's notice reads the stop report through the park marker, the reflection reads the friction directory, and a record per settle keeps caps across restarts and gives `report-timing` a settler to map onto its own categories.

### D10. A recorded parent decision relaunches a go-driven run through the go arm

Decision: batten's awaiting arm, which spawns a decided child through its bootstrap, lands on start's go arm and spawns `lyx loom drive`; the gate then reads the recorded decision.
`lyx loom resume` over a halted run that has no live driver (no driver strand, no run lock, no driver lock) and a seed recording `go` spawns `lyx loom drive` as start's go arm does, instead of refusing toward `lyx loom start`.
Resume's `Long` help (its opening statement that resume never spawns a driver, and the `lyx loom start` refusal branch) and that branch's refusal-spec row change with it.
Every other resume branch and refusal (pause, `merge_in_progress`, `driver_not_parked`, an awaiting run with no decision) is unchanged.
Rationale: a Go driver exits at every hand-back, so without this the orch's usual "decide, then resume" would always need a second command; the seed read is in loom's bootstrap verbs, where `PATTERN-driver-choice-single-site` allows it.

### D11. A pause holds the Go driver's spawns and re-steps

Decision: while a hub or task pause covers the slug, the Go driver spawns no tackler or arbiter, and performs no re-step, goto or hand-over; it waits in the listed pause hold and acts on the stop afresh when the pause lifts.
A tackler or arbiter already live when the pause lands is paused with the pair's other agents, as `lyx pause` treats every agent; the tackler's deadline excludes the paused interval (D7).
Rationale: a pause holds new agent and network work everywhere else, and the step loop already honours it.

### D12. A moved run is a new stop, never a park

Decision: before acting on a verdict or a rule, the Go driver re-reads the run's status.
A status file carries no reason, so the re-read classifies a `paused` status by its last history entry: an `OutcomeGoto` entry is a moved run, never an empty-reason `paused` stop (D13).
When another actor moved a halted run, or a verdict's re-step is refused as moved, it acts on the fresh state as a new stop instead of parking.
When the fresh state is `running`, or the run lock is held by another process, it writes a settle record and exits.
Rationale: batten, the orch or the operator may move a halted run while the Go driver waits on an agent; parking on that refusal left a run waiting for no reason in `arbiter-relaunch-moved`.

### D13. The empty-reason `paused` stop is found or reported

Decision: a reason lives only on the step result and the stop envelope; `shedengine.Status` carries none, and this decision adds no status field.
The empty-reason `paused` step stop is the designed outcome of two `shedengine` step paths, which return a `StepResult` with no `Reason`: the bare `pause_requested` pause and the cancelled context, the latter in both `stepLocked` branches (the pause check before the producer call and the cancellation after it).
Shed sets `StepResult.Reason` on each: `pause requested` on the bare pause, `cancelled` on a cancelled context, so every `paused` step stop carries one.
A goto is not a step stop: `Goto` persists `StatePaused`, but the next step passes the pause check and resumes, so a goto reaches the Go driver only through D12's status re-read, which reads it as a moved run by its `OutcomeGoto` history entry.
The Go driver writes a Go-authored friction note only for a `paused` step stop whose reason is still empty after that change, so the reflection files any path the change missed.
Rationale: the scope asks to fix or report it; a reason on each step path that lacked one, and goto classified by history rather than by reason, keep a routine outside goto from filing a note, the friction note is the sanctioned report path, and `PATTERN-agent-filed-issues` keeps issue filing with `lyx selfreport create`.

### D14. The arbiter receives the tackler's case brief, never its authority

Decision: on `escalate` the tackler writes a case brief with four fixed parts: the question the arbiter must answer; facts, each with a pointer (file and line, log path and line, or trace id); what it ruled out and how; its hypothesis, marked as one.
The arbiter's stencil gains a case-brief path marker, the literal `none` when absent, and lyx's own text telling the arbiter to check every fact it relies on and never take the hypothesis as given.
The arbiter's instructions, allowed actions and verdict format stay lyx's: the brief is an input file and changes none of them.
Rationale: the brief saves opus time on search while a wrong cheap diagnosis cannot steer or widen the verdict.

### D15. A go-driven run has no message receiver

Decision: the orch's role stencil gains one line: a go-driven run has no driver to message, so the orch addresses it through `lyx loom` and `lyx shed` verbs (pause, goto, decision, resume).
This departs from `orch-split`'s settled routing, where the run's driver receives every instruction for the run, and from `creel`'s planned per-run driver address; both apply to LLM-driven runs only until a later task gives go-driven runs a receiver.
Rationale: content for a run already has durable verbs, and the most urgent instruction, stop, is a pause, which needs no live receiver.

### Added after Discussion (parent, 2026-10-12): D15 accepted: a go-driven run has no message receiver yet

Decision: Accept D15: a run on the Go driver has no live receiver for run messages; until a later task adds one, a go-driven run is reached through lyx loom and lyx shed verbs. go-driver-assistant lands before judge-cap-call; judge-cap-call takes its arbiter-stencil, loom-config and loomcli/doc.go changes at Merge-In.

Rationale: orch-split still lists the receiver for a run with no long-lived driver as open, and the Go driver is opt-in per task while the LLM driver stays the default. The two runs touch the same files but their designs do not conflict; the run further along lands first.

## Constraints

- `PATTERN-driver-choice-single-site`: the recorded seed driver is read only in loom's bootstrap verbs (start, resume) and `lyx loom drive`'s own package; the background's permitted-consumer list gains Seed-Child's and loom's seed writer's `driver_kind` reads, in the same commit.
- `PATTERN-shed-verb-set`: `lyx loom drive` calls the `shedverbs` step-loop and goto bodies and reimplements neither; `lyx loom run` stays the generic body.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: every new key is classified in loom's `ConfigSpec` in the same change; `driver_kind` and the tackler keys are open to the local layer; Seed-Child reads loom config through `configengine`'s strict load.
- `PATTERN-spawn-observability`: the Go driver logs every tackler, arbiter and hand-over spawn, and the teardown where it waits; tests reach every spawn through a process seam.
- `PATTERN-daemon-wakeups`: the Go driver's unbounded waits are the listed loop waiter, shuttle's `Wait` and the pause hold; batten's binary check lives inside the listed Run-Shed wait.
- `PATTERN-shuttle-stop`: a tackler past its deadline is stopped only through shuttle's stop verb.
- `PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the driver lock and the tackler in-flight marker are ephemeral, the tackler attempts and settle records durable, all declared in `shedrun`.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`: the tackler stencil is read at call time from the told stencils directory, renders both directives, asks for no skill and sets no color.
- `PATTERN-friction-capture`: a Go-driven halt still writes its Go-authored friction note, as the go arm does today.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: every new or changed refusal (`drive` over a held driver lock, resume's changed branch, a typed `--child-driver` against an adopted seed that records none) names its way forward and has its refusal-spec row.
- Docs land in the same commit: `internal/loomcli/doc.go` gains Go-driver and tackler sections and its Start, Resume and Arbiter sections follow; `internal/battenshed/doc.go` follows for the driver lock, the binary re-step and the tackler hold; `docs/overview.md` follows if its execution stack names the driver.

## Auto-mode assumptions

- A1. The tackler lives in loom beside the arbiter, though the name fits any shed run: no other recipe needs a driver today.
- A2. The tackler has no CLI verb; the Go driver calls it in-process, as it calls the arbiter.
- A3. `driver_kind` is the key name, to keep it apart from the existing `driver` model-spec key.
- A4. Tackler defaults: `haiku`, 10 minutes, 4 spawns per run.
- A5. A tackler timeout or malformed verdict hands back to the parent rather than escalating to the arbiter.
- A6. The Go driver has no pane or strand of its own; the operator's run window shows the status strand and any tackler or arbiter strand.
- A7. The binary watch after a hand-back is batten's, so a standalone go-driven run with no batten has none.

## Open risks

- R1. Windows: a long-lived `lyx` process holds its executable, as batten and the hub daemons already do; the Go driver adds one per go-driven run while it settles.
- R2. A live tackler or arbiter strand meets `lyx pause`'s Esc and `lyx resume`'s typed line like any agent, which `pause-esc-rewind` reports can open the rewind picker; the arbiter's own deadline still runs through a pause.
- R3. A tackler strand can die at spawn from the pane squeeze `run-window-bugs` records; it reads as a `hand-back`, so the run reaches the orch rather than stalls, but at a cost until that task lands.
- R4. With a hub default of `driver_kind: go`, every new task switches drivers at once; the setting is meant per task first.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - binary-changed-field-vs-on-disk-build (class consistency, severity MEDIUM, open in rounds 2)
  - empty-reason-paused-is-by-design (class design, severity BLOCKING, open in rounds 1, 2)
  - tackler-bound-unenforced (class design, severity MEDIUM, open in rounds 1, 2)
  - tampered-check-vs-moved-run-rule (class consistency, severity MEDIUM, open in rounds 2)
  - verify-goto-record-match-and-retry-split (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Seeding: `run_settings` `loom.driver_kind: go` seeds the Go driver; no setting seeds the LLM driver; a typed `--child-driver` wins over both; a typed `--child-driver` against an adopted seed that records none is refused with its way forward; an invalid value routes Seed-Child to Stuck; a standalone `lyx loom start` honours the key; changing `run_settings` after seeding leaves the recorded driver unchanged.
- Each row of D3's table is covered, the stop class in and the action out, including the repair cap's third failure going to the tackler, and a tackler `escalate` at an `unseeded` error handing back because the arbiter refuses it.
- Verify failure halts: a watched run's halt is left to batten and sends the unwatched hand-back line; an unwatched run's first halt is moved and re-stepped, its second handed back; a halt with only a reason-text way forward goes to the tackler.
- A stale verify failure record from an earlier halt causes no goto, and its `blocked` stop goes to the tackler; a run whose retry batten spent is not retried by an unwatched Go driver, and a run whose retry the Go driver spent is not retried by batten, which reads the spent retry from the child's settle records.
- A tackler `resume` whose stop key changed between spawn and consume performs nothing and is handled as a new stop; an orch goto during a tackler wait is handled as a new stop with the verdict discarded, not as `tampered`; with the stop key unchanged, a tackler that moved HEAD, committed, switched branch, changed the uncommitted-path set or edited an already-dirty path reads as `hand-back` with cause `tampered`.
- The tackler's three verdicts are each acted on; each malformed case D7 lists reads as `hand-back` with its cause, a `goto` to a row before the halting entry's routed-from row reading as `precondition`; `tackler_max_spawns: 0` and `arbiter_max_spawns: 0` hand back without a spawn.
- `escalate` passes the case brief to the arbiter, whose prompt carries the brief path and the check-the-facts rule.
- Liveness: while a Go driver settles, start does not spawn a second one, resume refuses `driver_not_parked`, batten adopts it, and a second `drive` refuses; a running go child with no driver lock and no run lock is noticed as driver-dead.
- Restart: a Go driver killed during a tackler or arbiter wait spawns no second agent on restart and reuses the verdict; killed between a performed `resume` and its settle record, it reads the `applied` mark and escalates on a returning stop key instead of re-performing; caps survive.
- A run moved by another actor while the Go driver waits is acted on as a new stop; a run found running exits.
- A pause covering the slug holds every spawn and re-step until it lifts, and a tackler's paused interval does not count toward its deadline.
- Parent notification: watched and unwatched runs send exactly the D8 messages, the `parent_notice` relayed after an arbiter `hand-on`; no orch strand logs and does not queue.
- Every hand-back leaves a stop report, a park marker and settle records; a fresh `drive` removes the previous park marker; batten's stop notice carries the report and waits while a tackler is in flight.
- A decision recorded at an awaiting go child is acted on by batten through start's go arm; `lyx loom resume` over a halted, driverless go-seeded run spawns a Go driver; its other branches are unchanged.
- Binary change: a changed on-disk build at an eligible stop hands over once to a fresh driver; a dirty tree blocks it and names its paths in the stop report; a hand-over whose spawn fails leaves a parked run with a stop report; batten re-steps a halted driverless go child once per episode and build, never an LLM child, and never a halt its publish retry is taking.
- A bare `pause_requested` pause and a cancellation, in both `stepLocked` branches, each return a `paused` step stop with a reason; an outside goto read by D12's re-read is a moved run and writes no friction note; a `paused` step stop whose reason is still empty writes a friction note.
- Tests reach no LLM: shuttle, clock and process seams are faked, and the tackler stencil is checked by its render.

## Notes for the plan writer

- The go arm's spawn and handshake are in `internal/loomcli/start.go` and `mustSpawnDriver` in `internal/loomcli/bootstrap.go`; the LLM driver's stop handling is `contracts/stencils/shed/shed-template-driver.md` with its guide and two notify stencils.
  The rule table mirrors it except where D3 departs: a `blocked` stop and a transient that reaches the driver go to the tackler before any arbiter spawn, and `interrupted` under `handback` hands back with no fork.
- The arbiter's spawn path, eligibility, `applied` mark and records are in `internal/loomcli/arbiter*.go`; its stencil already carries a review-brief marker, which the case-brief marker sits beside.
- Batten's child-driver resolution is `childDriverOf` and `battenChildDriver` in `internal/battencli/arm.go` and the `ChildDriver` closure in `internal/battencli/wire.go`; the publish retry and its failure-record read are in `internal/battenshed/publishretry.go`; the notice hold is `arbiterHolds` in `internal/battenshed/notice.go`.
- The verify failure record is written by `internal/landingshed/verifygate.go` where a recorder is set.
- `binary_changed` is computed in `internal/shedverbs/steprecord.go`.
