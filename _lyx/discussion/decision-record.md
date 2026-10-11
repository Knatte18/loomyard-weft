# Decision record — burler-multi-llm

## Goal

Burler's review round runs as one seat table on the seats engine that `MultiLLMProducer` runs on: the fixer is the `chair`, the reviewer is `advisor-1`.
The ad hoc two-agent wiring (`roundHalves`, the parallel `join`, `lyx burler await-review`) goes.
The round's outcome, review files, fixer-report, failure rules and Bouncer contract stay as they are.
The chair keeps its prompt cache warm while it waits, at least as well as the fixer does today, and a seats chair's planned waits stop reaching the parent as stall notices.

## Scope

In:

- `internal/burlerengine` builds a `seatengine.Table` per round and runs, probes and resumes it through `seatengine`, replacing its own two-run start, join and probe.
- `internal/seatengine` gains the generic seat capabilities the round needs: a per-advisor gate, per-seat fork authorization, a required advisor, an acceptance hook on a required advisor, the advisor's terminal shuttle result on its `SeatResult`, and deferral of the chair's held-turn notice while advisor work is outstanding.
- `internal/shuttleengine` gains a told hold-notice deferral on `Spec`, the mechanism the chair deferral rides.
- The two Burler orchestrator stencils become the chair and advisor seat stencils, including the seat directives; the fixer waits with `lyx shuttle await`.
- `lyx burler await-review` is removed, and the run-wide `loom.yaml` key `fix_start` with its `after-review` value is retired.
- Every caller of the burler engine moves with it: `internal/shedadapters.BurlerProducer` (every review segment, Darn's perch included) and `lyx burler run`.
- Docs in the same commit: `doc.go` of `burlerengine`, `seatengine`, `shuttleengine` and `burlercli`, `docs/overview.md` where it names the round's wiring or `await-review`, and `tools/sandbox/SANDBOX-BURLER-SUITE.md`.

Out:

- The Shed row: Burler rounds stay on the `BurlerProducer` row; no recipe row changes producer type.
- `MultiLLMProducer`, the Discussion seats table, the chair stencil wording shared with Discussion beyond the hold deferral, and `twill`.
- Lens advisors: a fan's lenses stay forks of the reviewer seat.
- The review format, verdict parse, rubric, focus directive, finding classes, Bouncer prompt, judge, convergence rule and round artifact names.
- The parent-review cold-cache case (board note `parent-review-cold-cache`).

## Decisions

### D1 — Burler stays its own Shed row and drives the seats engine underneath

Decision: `BurlerProducer` keeps resolving the round, hydrating prior rounds, retrying a died or timed-out round and handing a completed round to the Bouncer.
`burlerengine.Engine.Run`, `ProbeRound` and `Resume` keep their signatures' roles and build one seat table per round, run through a `seatengine` seam (`Run`, `Probe`, `Resume`) instead of two shuttle runs.
`MultiLLMProducer` is not placed in the Burler row.
Rationale: everything Burler-specific (round resolution, hydration, retry, the Stuck hand-off, verdict parse, cluster audit, the failure rules) lives above or beside the two agents; only the agent wiring is what the seats engine replaces.
`MultiLLMProducer` maps a chair result onto the Shed contract generically and would have to grow every Burler rule to replace `BurlerProducer`.

### D2 — The seat table

Decision: role prefix `burler`, so the strands are `<shortname>:<slug>:burler-chair` and `<shortname>:<slug>:burler-advisor-1`; the role names are formed through `seatengine.SeatRole` and owned by `burlerengine`.

| Seat | Agent | Stencil | Model | Outputs | Gate | Forks |
|---|---|---|---|---|---|---|
| `chair` | fixer | `burler-template-fix-orchestrator` | `RunOpts.Fix` | fixer-report | `RunOpts.Gate`, each entry wrapped by `repairReportBeforeGate` (the table gate) | never |
| `advisor-1` | reviewer | `burler-template-review-orchestrator` | `RunOpts.Review` | review file | `ReviewGateEntry` (the seat gate) | when the profile names a cluster fan |

The review file is among the chair's inputs, as the table rules require.
Both seats load `burlerSkills`, run in the review segment's color, carry the round's number and share `RunOpts.Timeout`.
The step instruction files are still materialized per round under `.lyx/burler/round-*` and their absolute paths reach the seats as marker values.
`burlerengine.Geometry` gains `Shortname` and `Slug`, filled by `hubgeom` and `standalonegeom`, since the seats engine forms and checks strand names from them.
The stencil registry entries in `contracts/stencils/stencils.go` that key the orchestrators by `burler-review` and `burler-fix` move to the new role names.
Rationale: the board fixes the seat assignment; the existing orchestrators already split per agent, so they become the seat stencils with no new stencil file.

### D3 — Review-before-fix stays a Go gate: the required advisor and its acceptance hook

Decision: `seatengine.Seat` gains `Required` and `Accept`, both valid only on an advisor, `Accept` only on a `Required` seat (`Table.Validate` refuses anything else as a loud error naming the seat).
When a required advisor's run ends done with its gate passed, the engine calls `Accept` with that terminal shuttle result exactly once, in the advisor's waiting goroutine.
Burler's `Accept` is today's `reviewReady`: cluster audit, strict parse, then the ready marker write, recording the released fact as the write begins.
A required advisor that ends in anything but done, whose wait errors, or whose `Accept` errors, ends the step: the engine stops the chair and every other started advisor and returns, with no failure line queued to the chair.
Bound: only a seat marked `Required` can end the step early, and only through its own end or its own `Accept`; the Discussion seats table marks no seat, so its behavior is unchanged.
Rationale: PATTERN-review-round requires the fixer to touch nothing before Go has accepted the review and written the marker; a seats advisor's death is otherwise advisory, which would leave the fixer waiting to its timeout.

### D4 — Burler's failure rules map onto the seat result unchanged

Decision: `seatengine.SeatResult` gains the advisor's terminal `shuttleengine.Result` and its `Accept` error.
`decideRound` stays the one place the round's rules live and reads the seat result:

- the reviewer ended non-done or its wait errored: the round returns the reviewer's outcome with a nil error; an audit or strict-parse error from `Accept` is returned as today;
- the fixer died or timed out while the reviewer ran: the round returns the fixer's outcome;
- the fixer reached done before the marker was released (the recorded fact, never arrival order): the skipped-handoff error;
- otherwise the fixer's run decides; a failing fixer gate passes through with Verdict and Findings empty, and a done fixer has the review file compared against the accepted bytes before Verdict and Findings are set.

`Result.Review` and `Result.Fix` keep their `Half` fields (session id, strand guid, last message, run dir, start error, times, usage), the reviewer's taken from its seat result; `ForkAudit`, `Lenses`, `ClusterWarnings`, `Gate` and `NotStarted` keep their meaning.
A seat stop failure surfaces as `ErrSeatNotStopped` with its way forward; `ErrHalfNotStopped` is replaced by it or wraps it so `BurlerProducer`'s refusal keeps naming the way forward.
Rationale: the board keeps the round's outcome and the Bouncer contract; the rules are Burler's, the seat engine only reports what each seat came to.

### D5 — Start order is the seats engine's: chair first; `fix_start` is retired

Decision: the chair (fixer) starts first and the reviewer second, as every seat table does.
A chair that fails to start starts no reviewer: the round is `OutcomeDied` with `NotStarted` and `Fix.StartError`.
A reviewer that fails to start is a required-advisor failure: the chair is stopped and the round is `OutcomeDied` with `NotStarted` and `Review.StartError`.
The `loom.yaml` key `fix_start` and the `FixStart` run option with its `after-review` value are removed; the template drops the key, and a hub file still carrying it is reconciled as a removed key and loads.
Rationale: the hub runs `parallel`; `after-review` would need the seats engine to defer the chair behind an advisor, against the chair-first rule `seats-chair-first` just landed, for a knob no hub uses.

### D6 — Lenses stay forks of the reviewer seat

Decision: `seatengine.Seat` gains `ForkSubagents`, passed to that seat's shuttle spec; Burler sets it on the reviewer exactly when the profile names a cluster fan, and never on the chair.
The cluster audit (exact-N forks, no named spawn, no fork writes or Agent calls) runs in `Accept` on the reviewer's done result, as today; `fan_review` keeps setting the reviewer's model and so the forks'.
Rationale: forks read the reviewer's exploration from cache, while seats advisors re-read the code at full price (first Discussion bench, `docs/research/discussion-bench.md`); the fan's consolidation and audit machinery stays untouched.

### D7 — The chair waits on the ready marker with `lyx shuttle await`

Decision: the fixer's chair stencil runs `lyx shuttle await <ready-marker>` with the Bash tool's `timeout` at 300000 and repeats it until `ready: true`, editing nothing before then, as it does today with `await-review`.
`lyx burler await-review` and `internal/burlercli/await.go` are removed; `burlermarker.Path` still derives the marker path.
Rationale: the board requires the general capped wait `twill` built; its 250 s default cap keeps every re-run inside the 5-minute cache lifetime, so the chair stays warm exactly as the fixer does today, and a chair that waits inside its turn takes no held turn end.

### D8 — The seat channel inside a round

Decision: the chair may message the reviewer only after the ready marker exists, and only to ask what a finding means or rests on; an existing marker stands in for the reviewer's up line, which may never land on a chair that waits inside its turn.
The reviewer sends the chair nothing unprompted, answers only by message and never edits its review once written; the engine's byte comparison after the fixer is done stays the check.
A reply is advice: the fixer still validates every finding against the code and the fasit, and a dispute still needs cited evidence in the fixer-report.
These rules live in the Burler seat stencils; the shared seat directives are included unchanged.
Bound: the channel lets the fixer learn a finding's intent, never change the accepted review or bypass the dispute rules; it opens only after Go accepted the review, so the reviewer's judgment is fixed before the fixer speaks.
Rationale: the seat table gives the channel for free; scoping it after acceptance keeps the review independent.

### D9 — A required advisor keeps its hold notice

Decision: a `Required` advisor's shuttle spec leaves `QuietHold` false; other advisors stay quiet.
Rationale: today a reviewer that ends a turn without its review notifies the parent; the reviewer is the round's critical path, so silencing it would hide a stalled round until the timeout while the chair waits inside its turn.

### D10 — The chair's held-turn notice is deferred while advisor work is outstanding

Decision: `shuttleengine.Spec` gains a told deferral predicate for the held-turn notice.
For a run that sets it, a held turn end is notified only once the run has stayed held for a fixed settle interval and the predicate then reports no outstanding work; the wait loop re-checks a held, unnotified turn end on its existing poll, and a turn end the run has since moved past is never notified.
Runs without a predicate notify as today.
`seatengine` sets the predicate on the chair only: advisor work is outstanding while any started advisor's up line has not landed, or any started advisor's session is mid-turn by shuttle's session-state fold (a turn start with no turn end after it).
The seat handle exposes that reading from the advisor's run; no decision reads `@lyx_wait` (PATTERN-wait-mark-display-only).
Bound: the deferral delays a chair's notice, never drops it: once no advisor is mid-turn and every up line has landed, a chair still held past the settle interval notifies once; an advisor that stays mid-turn defers it at most until that advisor's own deadline, the table timeout.
The settle interval is one constant in `shuttleengine`, short enough to notify a real stall within a minute and long enough to cover a SendMessage's delivery to an idle advisor.
Rationale: lyx cannot see Claude Code's inbox, so an "unanswered batch" is read as an advisor that is working on it; the settle interval covers the instant between the chair sending a batch and the advisor starting its turn, and a chair whose advisors are all idle is a real stall.

### D11 — Resume keeps today's rule

Decision: `ProbeRound` runs the seats engine's probe for the round's table, and `Resume` attaches only when the chair and the reviewer are both live: it starts nothing, removes a ready marker it finds, and joins both as a fresh round does.
Any other combination is returned for `BurlerProducer` to settle by stopping what is live and running a fresh round, as today; a done reviewer kept by its pane is removed by the probe and counts as not live.
Rationale: a finished reviewer's run was never live today either, so a resumed round behaves as before.

## Constraints

- PATTERN-review-round: the fixer touches nothing before Go accepted the review and wrote the marker (D3, D7), and the review file is compared after the fix (D4).
- PATTERN-told-geometry: `burlerengine` and `seatengine` derive no path; the new geometry fields are told by `hubgeom` and `standalonegeom` (D2).
- PATTERN-agent-name: role names formed through `agentname` via the seats engine; `burlerengine` owns the `burler` prefix (D2).
- PATTERN-parent-directive and PATTERN-edit-directive: each seat stencil renders the parent directive and the edit directive exactly once, whether through the Burler stencil or the seat composition.
- PATTERN-role-skills-typed and PATTERN-spawn-color: skills and the review segment ride the seat specs; no stencil asks to load a skill or set a color.
- PATTERN-stencil-ownership and PATTERN-producer-pointer-rule: stencils read at call time; the Burler stencils point at the seat directives and never paraphrase them.
- PATTERN-wait-mark-display-only: the hold deferral reads session state, never the wait mark (D10).
- PATTERN-daemon-wakeups: the deferral re-check rides the wait loop's existing poll; no new unbounded wait.
- PATTERN-shuttle-provider-seam: the deferral and the session-state reading are provider-neutral; nothing Claude-specific leaves `claudeengine`.
- PATTERN-orch-pane-single-writer: the deferred notice still goes through the runner's notifier.
- PATTERN-config-strictness and PATTERN-binary-change-reconcile: retiring `fix_start` goes through the template and `hubreconcile.Ensure`, never a hand edit (D5).
- PATTERN-sandbox-coverage: the burler module stays exercised by the sandbox suite after the CLI verb is removed.
- PATTERN-test-economy, PATTERN-test-speed, PATTERN-testkit: unit tests drive fakes at the module surface; a fake seat runner shared across packages lives in one testkit kit; llm-tier smoke tests stay in their own files.
- Every Burler caller and every review segment (Discussion-Review, Plan-Review, Webster-Review, Darn's perch) keeps its round artifacts, paths and Bouncer hand-off byte-for-byte.

## Auto-mode assumptions

- "Runs its round on `MultiLLMProducer`" means the seats engine `MultiLLMProducer` is built on, not the producer type itself (D1).
- `fix_start` / `after-review` is retired rather than carried into the seats engine (D5).
- The chair may question the reviewer after acceptance (D8); the board neither asks for nor forbids it.
- The strand names change to `burler-chair` and `burler-advisor-1`, following the board's "chair" and "advisor-1" (D2).
- An advisor's mid-turn state is readable from shuttle's session-state fold, including turns a SendMessage starts (D10).
- The settle interval is a constant, not a config key (D10).

## Open risks

- A SendMessage that wakes an idle advisor may not fire the provider's turn-start hook; then an answering advisor reads as idle and the chair's notice fires after the settle interval. The llm-tier smoke or a watched run must confirm the turn-start signal.
- A fixer mid-round across the deploy calls the removed `lyx burler await-review` and fails; the round then ends on the skipped-handoff error or the timeout, and the row's retry or a re-step recovers. Deploy when no Burler round is in flight to avoid it.
- The reviewer's pane stays alive (kept) through the fix, so a round holds two panes until it ends.
- Starting the chair before the reviewer means a reviewer that never comes up costs one started-and-stopped fixer.
- The fork audit must still be attached to a kept advisor's done result; a shuttle path that skips it for `KeepPane` runs would fail every cluster round loudly.

## Acceptance criteria

- A round runs as one seat table: the fixer under `burler-chair`, the reviewer under `burler-advisor-1`, with the models, skills, segment, timeout, round number, outputs and gates of D2.
- The reviewer's review-parse gate re-prompts the reviewer in its own session; an unparseable review after the budget fails at the strict parse in `Accept`.
- The ready marker exists only after the reviewer's done result passed the cluster audit (cluster rounds) and the strict parse; a stale marker is removed before the round starts.
- Every failure rule of D4 holds: reviewer died or timed out (chair stopped, reviewer's outcome, nil error); audit or strict-parse error (error, chair stopped); fixer died or timed out while the reviewer runs (reviewer stopped, fixer's outcome); fixer done before release (skipped-handoff error); review changed after the fix (error); success (verdict, findings, gate passthrough, both halves' identities, times and usage).
- Not-started cases of D5: chair not started starts no reviewer; reviewer not started stops the chair; both report `NotStarted` with the right half's start error.
- A seat that cannot be stopped returns the not-stopped error with its way forward, and `BurlerProducer` surfaces it.
- Resume per D11: both seats live attaches and removes a found marker; any other live combination is returned for a fresh round.
- A cluster round sets fork authorization on the reviewer only and enforces the exact-N fork audit; a solo round authorizes no forks.
- `Table.Validate` refuses `Required`, `Accept` or a seat gate on the chair, and `Accept` on a non-required advisor.
- A Discussion seats table with no required advisor runs as before apart from the chair's hold deferral.
- Hold deferral: a chair held while an advisor is mid-turn or its up line is pending sends no notice; once the advisors are idle and the chair is still held past the settle interval it sends exactly one; a chair that starts a new turn first sends none for the old turn end; advisors and runs without a predicate notify as before; a required advisor's held turn end notifies the parent.
- The fixer's stencil names `lyx shuttle await` and no stencil, doc or code names `lyx burler await-review`; the verb is gone from the CLI.
- `fix_start` is gone from the template and loader, and a hub `loom.yaml` still holding it reconciles and loads.
- `BurlerProducer`'s round artifacts and Bouncer hand-off are unchanged for every review segment.

## Notes for the plan writer

- Burler's round code: `internal/burlerengine/engine.go` (`roundHalves`, `Run`), `handoff.go` (`reviewReady`, `join`, `decideRound`), `resume.go`, `shuttle.go`; the seats engine: `internal/seatengine/engine.go`, `table.go`, `prompt.go` (`QuietHold`/`KeepPane` per seat), `notice.go`, `resume.go`.
- The held notice: `internal/shuttleengine/hold.go` (`notifyHeld`) and the wait loop in `wait.go`; session state in `sessionstate.go`.
- The fixer stencil's wait lines are in `contracts/stencils/burler/burler-template-fix-orchestrator.md`; `contracts/stencils/claims_test.go` and `internal/burlerengine/prompt_test.go` pin `await-review` today.
- `lyx shuttle await` landed in 50cf6a425 (`internal/shuttleengine/await.go`).
- `fix_start` lives in `internal/loomengine/config.go` and `template.yaml`; its consumer passes `RunOpts.FixStart`.
- New top-level tests may push `burlerengine`, `seatengine` or `shuttleengine` past `cmd/lyx/testdata/test-budget.yaml`; a card that does lists that file under `Edit:`.
