# Decision record — driver-arbiter

## Goal

A run's driver that reaches a stop it cannot settle spawns a cold arbiter agent on a stronger model, which decides the stop from the task's own records, so the driver's parent (today the hub orch) is woken only for a call that needs an operator decision or that the arbiter is unsure of.

## Scope

In:

- The arbiter verb, role, stencil and config keys (D1, D3, D5, D7, D8, D11), for loom and darn runs alike, since both share the shed driver stencil.
- The driver stencil's arbiter branch (D6).
- Batten's notice hold (D10).
- `lyx loom decision add --by arbiter` (D12).
- The orch role stencil's escalation and stop sections (D13).
- Runs no batten watches get the arbiter too, since the driver stencil is the same; only the batten notice hold does not apply to them.
- Docs per CLAUDE.md: `doc.go` of each touched package, the `loom.yaml` template comments, `contracts/specs/refusal-spec.md` rows for the verb's refusals, and `docs/overview.md` if its module or execution-stack table changes.

Out:

- Parent review and PR-Gate: they stay with the orch, since they exist to bring in its context.
- A `paused` stop, an `awaiting` stop other than a Bouncer escalation (PR-Gate among them), a `busy` or `ownership` error, an `unseeded` error, a no-kind refusal, an `interrupted` stop under `interrupt_policy: handback`, and a `done` stop: none spawns the arbiter.
- A `blocked` stop at Publish whose verify-failure record names a target, in any run: in a batten-watched run batten's own one-shot Publish retry owns it, and a second failure keeps today's notice to the parent, since the verb cannot tell the first failure from the second; an unwatched run cannot be told from a watched one whose marker is missing (D2), so it keeps today's escalation too.
- Webster's own recovery strand, and the Bouncer's auto-accept: unchanged.
- The Go driver (`lyx loom run`): it never invokes the verb.

## Decisions

### D1 — The driver spawns the arbiter through one loom verb

Decision: a new verb, `lyx loom arbiter` (exact name the Plan's), run by the driver as a background job from its own cwd with the run id and the stop envelope's path, spawns the arbiter as a shuttle run in the task pair's reed session, waits for its verdict file, and prints one envelope carrying the verdict.
Its cobra `audience` keeps it out of the orch's command index, since only the driver runs it.
The driver stencil stays recipe-blind: the command reaches it through a new marker filled by `internal/loomcli`'s driver prompt, the same way `park_command` does.
An attempt is keyed to the stop it arbitrates by the stop key: the run's `current_producer`, `history_length` and `state`.
The verb also records the run's `pause_before` and `pause_after` in the attempt at spawn, for D6's state check.
An attempt's `resume` verdict is applied when the driver's re-launch consumes it (D6), which writes an applied mark into the attempt directory before it performs the verdict's goto or steps the run.
A re-run of the verb for the same stop key spawns nothing: it waits on a live arbiter of that attempt, returns a finished attempt's verdict while that verdict is unapplied, and refuses with a hand-on once an applied `resume` attempt holds that key (D7).
Rationale: the board entry names the driver as the spawner; a Go verb keeps eligibility, budget and the spawn deterministic and testable, and keeps the driver's context small as the loop verb does.

### D2 — Which stops are eligible

Decision: eligible, a closed list:

- `awaiting` at a Bouncer row whose current round carries a Bouncer-written escalation with no circling decision recorded (a spent budget or CIRCLING that the perch's auto-accept did not settle);
- `blocked`, except the Publish halt Scope excludes;
- a `producer`, `bootstrap` or `interrupted` (under `interrupt_policy: reinvoke`) error stop whose repair fork named escalation as its action, and a transient error that failed twice.

Go checks what the status file and envelope carry: the state, the row, the escalation and decision files, the Publish row's verify-failure record, `kind`, `transient` and `interrupt_policy`; any stop outside the list is refused with a way forward naming today's hand-back.
Whether the repair fork escalated is not in either file, so that half is a stencil-only bound: the driver stencil runs the verb only after a fork's escalation.
Bound: a driver that runs the verb at a `producer`, `bootstrap` or reinvoke `interrupted` error without a fork escalation skips the fork's diagnosis and the repair cap, and an arbiter `resume` then re-launches the run over them, first moving it back when the verdict names a goto target (D6); this is bounded by the spawn budget (D7), by the applied-attempt refusal that hands on a stop recurring after a `resume` (D1, D7), and by the arbiter having no repair verbs (D3), so the most it adds is one re-launch per budget slot.
A verdict's goto changes the stop key, so a stop recurring after it escapes the applied-attempt refusal, and the budget alone bounds it.
The Publish halt whose verify-failure record names a target is refused in every run, watched or not: an absent batten-watched marker is what both an unwatched run and a batten whose `MarkWatched` failed leave, so the verb cannot read a run as positively unwatched, and admitting it on an absent marker would race batten's own goto and resume at that halt.
The verb therefore never reads the watched marker.
Rationale: these are the stops whose way forward the driver cannot take and that wake the parent today.
Eligibility keys on the escalation, never on `parent_notice` alone, so a future parent-review escalation (`parent-review-wait`) cannot reach the arbiter.
An `interrupted` stop under `handback` stays out because a live agent may still run.

### D3 — What the arbiter may do itself

Decision: the arbiter reads anything read-only and may run exactly these mutating verbs in the task worktree:

- `lyx loom circling accept <slug>` or `lyx loom circling continue <slug>`, at a Bouncer escalation;
- `lyx loom decision add <slug> --by arbiter …`, to record a design call.

It never runs `lyx loom goto`: a `resume` verdict may instead name an earlier row as its goto target, which the driver's re-launch performs after its state check (D6).
It has none of the repair fork's repair verbs: at an error stop it may only judge that a plain re-launch is safe, name a goto target, or hand on.
A refusal from one of its verbs makes it hand on, quoting the refusal.
It then writes its verdict, `resume` or `hand-on`, with a reason, the verbs it ran, for `resume` the goto target when it names one, and for `hand-on` the question it hands on.
It never resumes or moves the run, never runs `lyx loom approve`, `reject` or `review`, never edits a file other than its verdict, never runs a mutating git command, and never messages any session.
Bound: the arbiter can end a review segment unconverged, grant one more round, append a dated, labelled decision-record entry, or name an earlier row for the re-launch to move a halted run back to; each is a record the orch and operator can already make, each is committed, and the run moves only when the driver re-launches the loop, which refuses a forward target as `lyx loom goto` does.
`circling`'s doc comment, which bounds the unchecked verbs partly by which stencils name them, gains the arbiter stencil beside the Bouncer escalation brief, bounded by D2's Bouncer-escalation eligibility and the verbs' own preconditions.
Rationale: the board entry leaves the split open; recording-only verbs plus a driver-applied re-launch keep the driver the sole loop owner, so no second process resumes a run concurrently.

### D4 — Hand-on criteria

Decision: the arbiter hands on when the call needs an operator decision that neither the board entry nor the decision record (with its support log) records, when it would widen the task's scope, or when it is unsure; otherwise it settles the stop.
For a review escalation it follows the orch's fixed rule: accept unless a finding makes the design wrong, and hand on a blocker that remains after round 3.
Rationale: the board entry's own rule for what reaches the parent.

### D5 — The arbiter's inputs

Decision: the arbiter's stencil is filled with the run id, the task worktree, the stop envelope's path, the driver's stop report or fork report path when one exists, the Bouncer escalation brief path when the stop is a review escalation, and its verdict path.
The stencil directs it to `lyx board get` for the task's entry, the decision record and support log, the latest review rounds the escalation brief names, and the stop's error.
It starts cold, with no fork of the driver's context.
Rationale: the board entry's input list; a cold start is what makes a stronger model affordable here.

### D6 — Driver handling of the verdict

Decision: Go validates the verdict before the verb returns it, and reads it as `hand-on` when it is missing or unparseable, or when a `resume` lacks its precondition:

- at a Bouncer escalation, a circling decision recorded for the round;
- at a `blocked` stop, the verdict names a goto target the re-launch admits.

An error-stop `resume` needs no recorded verb, and may name a goto target.
The driver then acts:

- `resume`: the driver re-launches the loop at once, naming the attempt (flag name the Plan's), and the run must move.
  The re-launch, in Go, takes the run lock before it consumes the attempt, so a busy refusal leaves the attempt unapplied.
  It then consumes the attempt: it refuses an attempt whose verdict is not `resume` or is already applied;
  then, in one status-file write under one hold of the status lock, it checks that the run is in the state the arbiter left, writes the applied mark (D1), and performs the verdict's goto when it names a target.
  It steps the run after that hold is released, since the step's own persists take the status lock.
  The run is in the state the arbiter left when its stop key is the attempt's, `pause_requested` is unset, and `pause_before` and `pause_after` hold the values the verb recorded in the attempt at spawn.
  The check runs before the goto, so no goto has yet cleared a pause set during the arbiter.
  The re-launch's goto is `lyx loom goto`'s backward move, except that it keeps `pause_requested`, `pause_before` and `pause_after` as they stand, so it removes no pause.
  Any other state, an orch or operator `goto` or `pause` (plain, `--before` or `--after`) among them, is refused before the goto, and the driver takes the `hand-on` path below with the verdict path; neither the goto nor the step happens, so a `resume` verdict never un-pauses a stop a human set.
  A re-launch steps an `awaiting` or `paused` run as it stands, since a step short-circuits only a `done` run and re-calls `current_producer`.
  This is a new entry in the stencil's closed re-launch list.
  It reverses, for this one case, the stencil's rule that a `blocked` stop is never repaired, and it is allowed over a spent repair cap; both are bounded by the spawn budget and by D7's same-stop rule.
- `hand-on`, a refusal for a spent budget, a timeout, or any verb failure: the driver writes its stop report with the arbiter's reason, question and verdict path, parks, and notifies its parent exactly as today, the parent notice relayed verbatim at an awaiting stop.

The stencil's sentence that `goto` is an operator and orchestrator verb the driver never runs stays true, and names the re-launch's goto of a consumed `resume` attempt as the one move the driver's loop makes on an arbiter's verdict.
On a wake that finds itself unparked with no loop job and an unapplied arbiter attempt for the current stop key on disk, the driver re-runs the verb, which returns that attempt's verdict (D1).
Rationale: the escalation path the parent already handles stays the one fallback, so the arbiter can only remove wakes, never lose a stop.

### D7 — Operator decision: a spawn budget in config, then the parent

Decision (operator, via `ly:orch`, 2026-10-11): the number of times one run may spawn the arbiter is a budget set in config, and once it is spent the driver escalates to its parent as the final step.
The key is `loom.yaml` `arbiter_max_spawns`, default `3` in the template; `0`, which a file lacking the key also reads, turns the arbiter off so every eligible stop escalates as today; a negative value is refused naming the key.
The count is the number of arbiter attempt records under the run's durable run directory, written when the verb spawns, so a driver or batten restart never resets it.
A stop that reaches the verb again with the stop key (D1) of an applied `resume` attempt is refused and handed on, so one recurring stop spends one slot and earns one re-launch.
This is the case at an error stop, where a producer error that recurs after the re-launch appends no history and leaves the stop key unchanged; the applied mark is what tells it from an unapplied verdict.
An applied attempt whose stop key is unchanged is handed on whether the stop recurred or the re-launch never stepped, as after a driver that died after writing the mark, or a step that failed before its first status write.
The refusal's reason states this as "an applied resume left the stop key unchanged" and never claims recurrence.
This costs the parent one wake and never a second re-launch.
Rationale: the operator set the shape; three lets a run settle a stop at each review perch while bounding a run that keeps getting stuck.

### D8 — Model and timeout keys

Decision: `loom.yaml` gains `arbiter` (model-spec, default `opus[high]`, where empty also takes `opus[high]`) and `arbiter_timeout_min` (default `30`, where `0` also takes `30`; a negative value is refused naming the key, as D7 refuses a negative budget).
A timed-out arbiter is stopped through shuttle's stop verb and reads as `hand-on` with the cause `timeout`; it has spent its budget slot.
Rationale: the driver runs `haiku[medium]`; the board entry asks for a stronger model such as `opus[high]`.
A present `loom.yaml` lacking a key reads its Go zero value (`config-layers-run-settings`), so each zero is given a safe meaning.

### D9 — Records and geometry

Decision: each attempt's prompt, verdict and envelope live under the run's directory in `_lyx` and are committed with the run records; the in-flight marker lives under the run's `.lyx` scratch directory at the mirrored subpath and records the arbiter's shuttle run and its deadline (spawn time plus the resolved timeout).
`internal/shedrun` declares the attempt directory as it declares the run's other paths.
Rationale: verdicts are the audit trail of decisions the parent did not make, and the budget count must survive a restart.

### D10 — Batten holds its notice while an arbiter is in flight

Decision: the InnerRun notice step treats a live in-flight arbiter marker for the child as "not ready", for a halted or awaiting child alike.
Live means the marker's arbiter shuttle run is alive and its recorded deadline plus the existing three-minute report wait has not passed; past that the notice goes as today.
Batten learns the marker's path from its caller, and reads the deadline from the marker, never from the pair's config.
When the arbiter's `resume` moves the child, the episode ends and no notice goes.
Rationale: without the hold, batten's three-minute report wait sends the parent a "report none yet" notice for every stop the arbiter is settling.

### D11 — The arbiter role in the spawn machinery

Decision: `internal/loomcli` spawns the arbiter and owns the role constant `arbiter`, formed by reed through `internal/agentname` into `<shortname>:<slug>:arbiter`.
Batten's strand revive at a halted child never revives an arbiter strand.
The arbiter's top-level stencil renders the no-parent variant of the parent directive: its verdict file is its only channel, and it takes no session's message as an instruction.
Its hand-on question is written for the driver's stop report; the stencil never asks it to address the operator.
Rationale: the arbiter answers through Go to the driver, which escalates; a parent the arbiter could message would be a second escalation path.

### D12 — `lyx loom decision add` admits the arbiter

Decision: the caller check admits a caller whose exported strand name, compared through `internal/agentname`, is the run's own arbiter strand, and only with `--by arbiter`; `--by arbiter` from any other caller is refused with a way forward.
Bound: the label stays unverified beyond the strand-name check, as `--by parent` is; the entry remains append-only, dated and committed.
Rationale: the check refuses task sessions today; an arbiter's design call is recorded where later producers and readers of the run see it, and after Plan approval it is an audit record that does not re-plan.

### D13 — The orch stencil

Decision: the orch role stencil's escalation and stop sections say that an escalation from a run whose arbiter handed on carries the arbiter's verdict path in the stop report, which the orch reads first instead of re-deriving the stop; the fixed review-escalation rule moves into the arbiter stencil and stays in the orch stencil only for a hand-on.
Rationale: the parent acts on the arbiter's question rather than repeating its work.
`orch-split` may move these sections to a run driver; the arbiter names only "the driver's parent", so it is indifferent to which agent holds that role.

### Added after Discussion (parent, 2026-10-11): The re-launch performs the goto, never the arbiter

Decision: Remove lyx loom goto from the arbiter's mutating verbs (D3); a resume verdict may name a goto target instead. The Go re-launch (D6) performs that goto itself under the status lock as part of consuming the attempt, and refuses it, taking the hand-on path, when pause_requested is set or pause_before/pause_after differs from the value recorded in the attempt at spawn. D6's blocked-stop precondition becomes 'the verdict names a goto target the re-launch admits'. Restate D2's and D3's bounds for the moved goto. Add an acceptance criterion: an orch pause (plain, --before or --after) during a live arbiter whose verdict moves the run back is handed on, and neither the goto nor the re-launch happens. Reword the Open risks bullet to match.

Rationale: shedengine Goto clears PauseRequested, PauseBefore and PauseAfter, so an arbiter goto erases a human pause before D6's check can see it; the check must run before anything clears the fields (round-2 F1).

### Added after Discussion (parent, 2026-10-11): A re-launch steps paused and awaiting runs as they stand

Decision: Delete D6's clause that the re-launch performs the un-pause lyx loom resume does; replace it with: a re-launch steps an awaiting or paused run as it stands, since a step short-circuits only a done run and re-calls current_producer. Mark the goto-paused question in the support log's Question ledger resolved by the code, and drop the Notes-for-the-plan-writer sentence asking the Plan to check it.

Rationale: lyx loom resume writes no status, it wakes the parked driver; stepLocked short-circuits only StateDone (round-2 F2).

### Added after Discussion (parent, 2026-10-11): An unchanged stop key after an applied attempt is handed on without claiming recurrence

Decision: In D7: an applied attempt whose stop key is unchanged is handed on whether the stop recurred or the re-launch never stepped; the refusal's reason reads 'an applied resume left the stop key unchanged' and never claims recurrence; this costs the parent one wake and never a second re-launch. In D2, add the Publish row's verify-failure record to the list of inputs Go checks.

Rationale: The applied mark is written before the run steps, so a busy refusal or driver death leaves the key unchanged without a step (round-2 F3); D2's Publish-halt exclusion reads that record (F4).

## Constraints

- PATTERN-stencil-ownership: the arbiter stencil is read at call time through `internal/stencilstore`.
- PATTERN-parent-directive and PATTERN-edit-directive: the arbiter stencil renders both shared directives.
- PATTERN-agent-name, PATTERN-spawn-color and PATTERN-role-skills-typed: the launch spec names the role constant, colour segment and skills.
- PATTERN-no-denied-recovery: no stencil or refusal names a command the agents' settings deny.
- PATTERN-refusal-way-forward: every refusal of the new verb names its way forward and lands a `refusal-spec.md` row and a test.
- PATTERN-cli-cobra: the verb carries `Command`, `RunCLI`, `audience`, `Short` and JSON errors.
- PATTERN-shuttle-stop: a timed-out arbiter is stopped only through shuttle's stop verb.
- PATTERN-completion-signal: shuttle's completion check consults the verdict file through `allOutputFilesExist`.
- PATTERN-pair-teardown: session end ends a live arbiter strand with the pair's other strands.
- PATTERN-daemon-wakeups: the verb's wait on the verdict wakes at most once a second and backs off.
- PATTERN-spawn-observability: the spawn and its teardown are logged through `internal/logger`.
- PATTERN-lyxdirs-single-declarer: no new file spells `_lyx` or `.lyx`.
- PATTERN-told-geometry: `internal/battenshed` is told the marker's location.
- PATTERN-driver-choice-single-site: the verb, D2's eligibility and D10's hold read no recorded seed driver; only the driver stencil invokes the verb.
- PATTERN-test-speed and PATTERN-test-economy: Go-tier tests with fake shuttle and clock seams; no `llm` test is needed.
- The driver stencil names no recipe and no recipe-owned command.
- Test budget: a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

Auto-picks, each with its rationale in its decision: the verb shape (D1), the eligible set (D2), recording-only arbiter powers with the goto moved into the re-launch (D3, D6), key names and defaults (D7, D8), the batten hold (D10), the no-parent directive (D11), unwatched-run coverage, the second-Publish-failure boundary and the Publish halt's exclusion in every run (Scope, D2), and the re-launch consuming the attempt (D6).

## Open risks

- The arbiter is on a stronger model than the driver but still an LLM: a wrong `circling accept` lets an unconverged segment pass; its verdict and the record entries it wrote are committed, so a reader can see who decided.
- An orch or operator `goto` or `pause` on the run while the arbiter runs is not blocked, but it is not overridden either: the re-launch checks the run against the attempt's stop key and recorded pause conditions before it performs the verdict's goto or steps the run (D6), and otherwise hands on, so such an action costs the arbiter's settlement of that stop.
- A driver that dies mid-arbiter and is never revived leaves the verdict unapplied until batten's hold expires and the parent is notified.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - regoto-gap-erases-plain-pause (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - arbiter-verb-unreadable-input-no-test (class scope, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- Each eligible stop kind of D2 spawns one arbiter, and each excluded stop kind, the target-naming Publish halt in a watched and an unwatched run and an awaiting stop with a `parent_notice` but no Bouncer escalation included, is refused with a way forward and spawns nothing.
- A re-run of the verb for the same stop key spawns nothing and returns the live or finished unapplied attempt's verdict.
- With `arbiter_max_spawns` spent, set to `0` or absent from the file, the verb refuses, spawns nothing, and the driver escalates to its parent as today; a negative value is refused naming the key.
- An absent `arbiter` or `arbiter_timeout_min` resolves to `opus[high]` and 30 minutes; a negative `arbiter_timeout_min` is refused naming the key.
- A stop that recurs unchanged after an arbiter `resume` is handed on without a second spawn.
- A producer error recurring at the same row after an applied arbiter `resume` is handed on without a second spawn and without a second re-launch.
- The spawn count survives a driver restart and a batten restart.
- A valid `resume` verdict leads to a re-launch that moves the run, over an `awaiting`, `paused` or `blocked` state alike; `hand-on`, a missing or unparseable verdict, a `resume` missing its precondition, and a timeout each lead to the park-and-escalate path with the arbiter's reason in the stop report.
- A timed-out arbiter is stopped through shuttle's stop verb.
- A driver that wakes unparked with an unapplied arbiter attempt on disk for the current stop key applies that attempt's verdict without a second spawn.
- An orch `goto` or an orch `pause` during a live arbiter, followed by a `resume` verdict, is handed on with the verdict path, not un-paused or re-launched, and spawns no second arbiter.
- An orch `pause`, plain, `--before` or `--after`, during a live arbiter whose `resume` verdict names a goto target is handed on with the verdict path, and neither the goto nor the re-launch happens.
- A plain `pause` that lands after the arbiter's verdict and before the re-launch's step survives the re-launch and pauses the run.
- An applied `resume` whose re-launch never stepped, such as after the driver died between the applied mark and the step, is handed on with the reason that the stop key is unchanged, without a second re-launch.
- Batten sends no state-changed notice while a live arbiter marker is within its deadline, sends it once past the deadline or after a hand-on, and sends none when a `resume` moved the child on; a marker whose arbiter shuttle run is dead, and a missing marker, hold nothing, and the notice goes as today; its strand revive never revives an arbiter strand.
- `lyx loom decision add --by arbiter` passes from the run's arbiter strand and is refused from any other caller; `--by parent` and `--by operator` behave as before.
- The arbiter stencil renders the edit directive and the no-parent directive, names only D3's mutating verbs, and the stencil tests over directives and role skills pass for the new role.
- The driver stencil stays recipe-blind and every marker is filled for loom and darn runs.

## Notes for the plan writer

- Precedents: `lyx webster recover-batch` spawns a cold strand through shuttle and reed (`internal/websterengine/recoverbatch.go`); `--until-stop`'s live-loop wait shows the "second invocation waits" shape; batten's `stateChangeReady` in `internal/battenshed/notice.go` is where the hold goes.
- `internal/loomcli/resume.go` notes that a step over an awaiting Bouncer row with a pending circling decision re-calls the Bouncer.
- `parent-review-wait` (running) and `orch-split` (note) also edit `contracts/stencils/orch/orch-template-role.md`.
