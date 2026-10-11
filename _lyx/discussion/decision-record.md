# Decision record — driver-arbiter

## Goal

A run's driver that reaches a stop it cannot settle spawns a cold arbiter agent on a stronger model, which decides the stop from the task's own records, so the driver's parent (today the hub orch) is woken only for a call that needs an operator decision or that the arbiter is unsure of.

## Scope

In:

- The arbiter verb, role, stencil and config keys (D1, D3, D5, D7, D8, D11), for loom and darn runs alike, since both share the shed driver stencil.
- The driver stencil's arbiter branch (D6).
- Batten's notice hold (D10).
- `lyx loom decision add --by arbiter` (D12).
- The orch role stencil's escalation and stop sections (D14).
- Runs no batten watches get the arbiter too, since the driver stencil is the same; only the batten notice hold does not apply to them.
- Docs per CLAUDE.md: `doc.go` of each touched package, the `loom.yaml` template comments, `contracts/specs/refusal-spec.md` rows for the verb's refusals, and `docs/overview.md` if its module or execution-stack table changes.

Out:

- Parent review and PR-Gate: they stay with the orch, since they exist to bring in its context.
- A `paused` stop, an `awaiting` stop other than a Bouncer escalation (PR-Gate among them), a `busy` or `ownership` error, an `unseeded` error, a no-kind refusal, an `interrupted` stop under `interrupt_policy: handback`, and a `done` stop: none spawns the arbiter.
- A `blocked` stop at Publish whose verify-failure record names a target, in a batten-watched run: batten's own one-shot Publish retry owns it, and a second failure keeps today's notice to the parent, since the verb cannot tell the first failure from the second.
- Webster's own recovery strand, and the Bouncer's auto-accept: unchanged.
- The Go driver (`lyx loom run`): it never invokes the verb.

## Decisions

### D1 — The driver spawns the arbiter through one loom verb

Decision: a new verb, `lyx loom arbiter` (exact name the Plan's), run by the driver as a background job from its own cwd with the run id and the stop envelope's path, spawns the arbiter as a shuttle run in the task pair's reed session, waits for its verdict file, and prints one envelope carrying the verdict.
Its cobra `audience` keeps it out of the orch's command index, since only the driver runs it.
The driver stencil stays recipe-blind: the command reaches it through a new marker filled by `internal/loomcli`'s driver prompt, the same way `park_command` does.
An attempt is keyed to the stop it arbitrates: the run's `current_producer`, `history_length` and `state`.
A re-run of the verb for the same stop waits on a live arbiter of that attempt, or returns a finished attempt's verdict, and spawns nothing.
Rationale: the board entry names the driver as the spawner; a Go verb keeps eligibility, budget and the spawn deterministic and testable, and keeps the driver's context small as the loop verb does.

### D2 — Which stops are eligible

Decision: eligible, a closed list:

- `awaiting` at a Bouncer row whose current round carries a Bouncer-written escalation with no circling decision recorded (a spent budget or CIRCLING that the perch's auto-accept did not settle);
- `blocked`, except the batten-owned Publish halt Scope excludes;
- a `producer`, `bootstrap` or `interrupted` (under `interrupt_policy: reinvoke`) error stop whose repair fork named escalation as its action, and a transient error that failed twice.

Go checks what the status file and envelope carry: the state, the row, the escalation and decision files, `kind`, `transient` and `interrupt_policy`; any stop outside the list is refused with a way forward naming today's hand-back.
Whether the repair fork escalated is not in either file, so that half is a stencil-only bound: the driver stencil runs the verb only after a fork's escalation.
A batten-watched run is recognised by the pair's watched marker; a stale marker only refuses the arbiter at the Publish halt, which falls back to today's escalation.
Rationale: these are the stops whose way forward the driver cannot take and that wake the parent today.
Eligibility keys on the escalation, never on `parent_notice` alone, so a future parent-review escalation (`parent-review-wait`) cannot reach the arbiter.
An `interrupted` stop under `handback` stays out because a live agent may still run.

### D3 — What the arbiter may do itself

Decision: the arbiter reads anything read-only and may run exactly these mutating verbs in the task worktree:

- `lyx loom circling accept <slug>` or `lyx loom circling continue <slug>`, at a Bouncer escalation;
- `lyx loom decision add <slug> --by arbiter …`, to record a design call;
- `lyx loom goto --to <producer>`, which already refuses a forward goto, a running run and a held run lock.

It has none of the repair fork's repair verbs: at an error stop it may only judge that a plain re-launch is safe, move the run back, or hand on.
A refusal from one of its verbs makes it hand on, quoting the refusal.
It then writes its verdict, `resume` or `hand-on`, with a reason, the verbs it ran, and for `hand-on` the question it hands on.
It never resumes the run, never runs `lyx loom approve`, `reject` or `review`, never edits a file other than its verdict, never runs a mutating git command, and never messages any session.
Bound: the arbiter can end a review segment unconverged, grant one more round, append a dated, labelled decision-record entry, or move a halted run back to an earlier row; each is a record the orch and operator can already make, each is committed, and the run moves only when the driver re-launches the loop.
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
- at a `blocked` stop, a goto recorded since the stop.

An error-stop `resume` needs no recorded verb.
The driver then acts:

- `resume`: the driver re-launches the loop at once, and the run must move: over an `awaiting` or `paused` state the arbiter's records left, the re-launch performs the same un-pause `lyx loom resume` does when a bare re-launch would not step it.
  This is a new entry in the stencil's closed re-launch list.
  It reverses, for this one case, the stencil's rule that a `blocked` stop is never repaired, and it is allowed over a spent repair cap; both are bounded by the spawn budget and by D7's same-stop rule.
- `hand-on`, a refusal for a spent budget, a timeout, or any verb failure: the driver writes its stop report with the arbiter's reason, question and verdict path, parks, and notifies its parent exactly as today, the parent notice relayed verbatim at an awaiting stop.

The stencil's sentence that `goto` is an operator and orchestrator verb the driver never runs stays true for the driver, and names the arbiter as the one other party that may run it.
On a wake that finds itself unparked with no loop job and an arbiter attempt for the current stop on disk, the driver re-runs the verb, which returns that attempt's verdict (D1).
Rationale: the escalation path the parent already handles stays the one fallback, so the arbiter can only remove wakes, never lose a stop.

### D7 — Operator decision: a spawn budget in config, then the parent

Decision (operator, via `ly:orch`, 2026-10-11): the number of times one run may spawn the arbiter is a budget set in config, and once it is spent the driver escalates to its parent as the final step.
The key is `loom.yaml` `arbiter_max_spawns`, default `3` in the template; `0`, which a file lacking the key also reads, turns the arbiter off so every eligible stop escalates as today; a negative value is refused naming the key.
The count is the number of arbiter attempt records under the run's durable run directory, written when the verb spawns, so a driver or batten restart never resets it.
A stop that reaches the verb again unchanged after an arbiter `resume` (same row and history length) is refused and handed on, so one recurring stop spends one slot.
Rationale: the operator set the shape; three lets a run settle a stop at each review perch while bounding a run that keeps getting stuck.

### D8 — Model and timeout keys

Decision: `loom.yaml` gains `arbiter` (model-spec, default `opus[high]`, where empty also takes `opus[high]`) and `arbiter_timeout_min` (default `30`, where `0` also takes `30`, floor `1` otherwise).
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

Auto-picks, each with its rationale in its decision: the verb shape (D1), the eligible set (D2), recording-only arbiter powers (D3), key names and defaults (D7, D8), the batten hold (D10), the no-parent directive (D11), unwatched-run coverage and the second-Publish-failure boundary (Scope).

## Open risks

- The arbiter is on a stronger model than the driver but still an LLM: a wrong `circling accept` lets an unconverged segment pass; its verdict and the record entries it wrote are committed, so a reader can see who decided.
- An orch or operator action on the run while the arbiter runs (a goto, a pause) is not blocked; the driver's re-launch then steps whatever state the run is in.
- A driver that dies mid-arbiter and is never revived leaves the verdict unapplied until batten's hold expires and the parent is notified.

## Acceptance criteria

- Each eligible stop kind of D2 spawns one arbiter, and each excluded stop kind, the batten-owned Publish halt and an awaiting stop with a `parent_notice` but no Bouncer escalation included, is refused with a way forward and spawns nothing.
- A re-run of the verb for the same stop spawns nothing and returns the live or finished attempt's verdict.
- With `arbiter_max_spawns` spent, set to `0` or absent from the file, the verb refuses, spawns nothing, and the driver escalates to its parent as today; a negative value is refused naming the key.
- An absent `arbiter` or `arbiter_timeout_min` resolves to `opus[high]` and 30 minutes.
- A stop that recurs unchanged after an arbiter `resume` is handed on without a second spawn.
- The spawn count survives a driver restart and a batten restart.
- A valid `resume` verdict leads to a re-launch that moves the run, over an `awaiting`, `paused` or `blocked` state alike; `hand-on`, a missing or unparseable verdict, a `resume` missing its precondition, and a timeout each lead to the park-and-escalate path with the arbiter's reason in the stop report.
- A timed-out arbiter is stopped through shuttle's stop verb.
- A driver that wakes unparked with an arbiter attempt on disk for the current stop applies that attempt's verdict without a second spawn.
- An orch goto during a live arbiter, followed by a `resume` verdict, re-launches over the moved state and spawns no second arbiter.
- Batten sends no state-changed notice while a live arbiter marker is within its deadline, sends it once past the deadline or after a hand-on, and sends none when a `resume` moved the child on; its strand revive never revives an arbiter strand.
- `lyx loom decision add --by arbiter` passes from the run's arbiter strand and is refused from any other caller; `--by parent` and `--by operator` behave as before.
- The arbiter stencil renders the edit directive and the no-parent directive, names only D3's mutating verbs, and the stencil tests over directives and role skills pass for the new role.
- The driver stencil stays recipe-blind and every marker is filled for loom and darn runs.

## Notes for the plan writer

- Precedents: `lyx webster recover-batch` spawns a cold strand through shuttle and reed (`internal/websterengine/recoverbatch.go`); `--until-stop`'s live-loop wait shows the "second invocation waits" shape; batten's `stateChangeReady` in `internal/battenshed/notice.go` is where the hold goes.
- `internal/loomcli/resume.go` notes that a step over an awaiting Bouncer row with a pending circling decision re-calls the Bouncer; check whether a bare re-launch also steps a run a `goto` left `paused`.
- `parent-review-wait` (running) and `orch-split` (note) also edit `contracts/stencils/orch/orch-template-role.md`.
