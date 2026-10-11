# Card 8 — burler-seat-table

**Create:**
- `internal/burlerengine#burlerRolePrefix`
- `internal/burlerengine#Profile.roundTable`
- `internal/burlerengine#roundAcceptance`
- `internal/burlerengine#roundAcceptance.accept`

**Delete:**
- `internal/burlerengine#burlerReviewRole`
- `internal/burlerengine#burlerFixRole`
- `internal/burlerengine#halfSpec`
- `internal/burlerengine#Profile.roundHalves`
- `internal/burlerengine#reviewerEnd`
- `internal/burlerengine#reviewerEnd.accepted`
- `internal/burlerengine#fixerEnd`
- `internal/burlerengine#fixerEnd.finished`
- `internal/burlerengine#Engine.stopHalf`
- `internal/burlerengine#Engine.join`
- `internal/burlerengine#awaitReview`
- `internal/burlerengine#Engine.probeHalf`

**Edit:**
- `internal/burlerengine#Engine`
- `internal/burlerengine#New` -> `func New(shuttle seatengine.Shuttle, geom Geometry, cfg Config, stencilsDir, frictionDir string) *Engine`
- `internal/burlerengine#Engine.Run`
- `internal/burlerengine#notStartedResult` -> `func notStartedResult(p *Profile, review, fix Half) Result`
- `internal/burlerengine#ErrHalfNotStopped`
- `internal/burlerengine#halfOf` -> `func halfOf(result shuttleengine.Result, strandGUID, runDir string) Half`
- `internal/burlerengine#Engine.decideRound` -> `func (e *Engine) decideRound(p *Profile, opts RunOpts, seats seatengine.Result, acceptance *roundAcceptance) (Result, error)`
- `internal/burlerengine#repairReportBeforeGate`
- `internal/burlerengine#Handle`
- `internal/burlerengine#LiveRound`
- `internal/burlerengine#Engine.ProbeRound`
- `internal/burlerengine#Engine.Resume`
- `internal/burlerengine#roundPrompts`
- `internal/burlerengine#composePrompt` -> `func composePrompt(stencilsDir string, p *Profile, patternDirective, frictionDirective string, paths roundFilePaths) (roundPrompts, error)`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlerengine/doc.go`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/burlercli/wiring_test.go`
- `internal/loomcli#loomCLI.wire`
- `contracts/stencils#roleOpeningStencils`
- `docs/overview.md`

**Uses:**
- `internal/burlerengine#reviewReady`
- `internal/burlerengine#ReviewGateEntry`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#Engine.Probe`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine#ErrSeatNotStopped`

**Intent:**
Run a Burler review round as one seat table on the seats engine (decisions D1, D2, D4, D5, D6 and D11): the fixer is the chair, the reviewer is `advisor-1`, and the ad hoc two-run start, parallel join and probe go.
The round's outcome, review files, fixer-report, failure rules and Bouncer contract stay as they are.

**The seam.**
`New` takes a `seatengine.Shuttle` and builds the engine's `seatengine.Engine` itself from the told geometry: `WorktreeRoot`, `AnchorPath`, `ParentName`, `Shortname` and `Slug` from `Geometry` (card 6) and `StencilsDir` from `stencilsDir`.
`Engine` holds it as a `*seatengine.Engine` field in place of its `Shuttle` field; the round's tests drive that real seat engine (shared decision `burler-builds-its-seat-engine`), so no interface stands between them.
Every caller of `New` passes `seatengine.RunnerShuttle(runner)` (`wireHub`, `wireStandalone`, `loomCLI.wire`, both smoke tests), which leaves burlerengine's own `Shuttle` and `RunnerShuttle` without a caller; card 12 deletes them.
`Handle` stays, in `shuttle.go`, as the type of `LiveHalf.Handle`; its doc says `seatengine.Handle` and `*shuttleengine.Run` satisfy it.

**The table (D2).**
`roundTable` builds the round's `seatengine.Table`: role prefix `burlerRolePrefix`, segment `segmentcolor.Review`, the table gate `opts.Gate` with every entry wrapped by `repairReportBeforeGate` (a fresh slice, as today), timeout `opts.Timeout`, round `opts.Round`, not interactive, and two seats in order.
The chair seat runs `burler-template-fix-orchestrator` on `opts.Fix`'s model, effort and version, reads the review path and writes the fixer-report path, loads `burlerSkills`, and never forks.
The `advisor-1` seat runs `burler-template-review-orchestrator` on `opts.Review`'s model, writes the review path, loads `burlerSkills`, is `Required` with `Accept` set to `accept`, carries the seat gate `ReviewGateEntry(p.ReviewPath)`, and sets `ForkSubagents` exactly when `p.ClusterFan` is set.
The strands are therefore `<shortname>:<slug>:burler-chair` and `<shortname>:<slug>:burler-advisor-1`, formed through `seatengine.SeatRole` by the seats engine, with burlerengine owning the prefix (`PATTERN-agent-name`).
Each seat's stencil markers ride the seat's values: the absolute instruction-file paths, `review_path`, `review_format_path` and `ready_marker_path`, as the two orchestrators name them today.
The seats engine renders the parent and edit directives into each seat exactly once: the chair's names the round's parent, the advisor's names the chair (`PATTERN-parent-directive`, `PATTERN-edit-directive`); skills and the segment ride the seat specs (`PATTERN-role-skills-typed`, `PATTERN-spawn-color`).
`composePrompt` stops filling the two orchestrators and drops its `parentName` parameter: `roundPrompts` carries each seat's marker values in place of the two filled strings, beside the four instruction files, which are still materialized per round under `.lyx/burler/round-*`.
Lenses stay forks of the reviewer seat; the cluster audit is unchanged (D6).

**Acceptance (D3).**
`roundAcceptance` holds the released flag (an atomic) and the accepted review's bytes, verdict, findings and cluster warnings.
`accept` runs today's `reviewReady` on the reviewer's done result: the cluster audit for a cluster round, the strict parse, then the ready marker write, the released flag set as the write begins; any failure writes no marker and is returned as the advisor's accept error.

**Run.**
`Run` keeps its sequence up to the prompts: validate, the PATTERN and friction directives, materialize the instruction files, then remove a stale ready marker before anything starts (D5: with the chair first, this removal is what keeps the fixer's wait from returning on a prior attempt's marker).
It then runs the table through the seats engine and decides the round with `decideRound`.
A chair that fails to start starts no reviewer: an error wrapping `shuttleengine.ErrNotStarted` is `OutcomeDied` with `NotStarted` and `Fix.StartError` and a nil error, and any other start error is returned wrapped as today.
A seat stop failure, `seatengine.ErrSeatNotStopped`, is returned wrapped in `ErrHalfNotStopped` beside it, so `BurlerProducer`'s refusal keeps matching and the message still ends with the way forward; `ErrHalfNotStopped`'s doc says so.

**decideRound (D4)** reads the seat result, the reviewer's from its `advisor-1` entry, and is still the one place the round's rules live:
- the reviewer failed to start: `OutcomeDied` with `NotStarted` and `Review.StartError`, the chair already stopped by the seats engine, nil error;
- the reviewer ended the step and its wait errored: that error wrapped as `burler: shuttle run`;
- the reviewer ended the step with an accept error (audit or strict parse): that error, with the reviewer's outcome and `ForkAudit`;
- the reviewer ended the step non-done: the reviewer's outcome, nil error;
- the reviewer ended the step done with its review gate failed: an error naming the review gate, never a verdict.
  No real reviewer reaches it, since `ReviewGateEntry` is `PassOnCap` and a gate outcome counts no `PassOnCap` entry against `Passed`; `decideRound` still maps every shape the seat result can take, and a fake reviewer whose done result carries `Gate.Passed` false reaches it;
- the chair's wait errored: that error wrapped as `burler: shuttle run`;
- the fixer died or timed out (it ended the step): the fixer's outcome, nil error;
- the fixer reached done while the released flag is false (the recorded fact, never arrival order): the skipped-handoff error with the fixer's outcome;
- otherwise the fixer's run decides: its `Gate` passes through, a failing fixer gate leaves Verdict and Findings empty, and a done fixer has the review file compared against the accepted bytes, a difference being today's error, before Verdict, Findings, `ForkAudit` and `ClusterWarnings` are set.
`Result.Review` and `Result.Fix` keep their `Half` fields through `halfOf`: the reviewer's from its seat result (terminal result, strand guid, run dir), the fixer's from the chair's result; `Lenses` as today.
Every seat is stopped by the seats engine before it returns, so no half is left live and no marker write outlives the attempt.

**Probe and resume (D11).**
`ProbeRound` builds the round's table with the same builder (instruction paths a round would be told, never created, since a probe composes no prompt) and runs the seats engine's probe.
A live chair is `Fix: HalfLive` with its handle, a found advisor `Review: HalfLive` with its handle; a seat not found is `HalfDone` when its output file exists and `HalfGone` otherwise, a done reviewer kept by its pane having been removed by the probe and counting as not live.
The seats engine's probe stops every live advisor when the chair is not live, so a live reviewer beside a gone fixer is stopped inside `ProbeRound` and reported `HalfDone` or `HalfGone` by its file; `BurlerProducer` then runs a fresh round, the outcome its stop-and-respawn of a lone live reviewer reaches today.
A probe stop failure, `seatengine.ErrSeatNotStopped`, is returned wrapped in `ErrHalfNotStopped`, as `Run` returns one, so `BurlerProducer` returns it bare and archives nothing.
`LiveRound` keeps its exported shape and gains an unexported `*seatengine.LiveTable` holding what the probe found, a pointer so the struct stays comparable.
`Resume` refuses, as today, unless both halves are live; it also refuses a `LiveRound` whose probed table is absent, which a caller reaches only by building a both-live `LiveRound` by hand instead of taking `ProbeRound`'s answer, as `shedadapters`' fake-runner tests do, and it then starts and stops nothing.
Otherwise it starts nothing, removes a ready marker it finds, runs the seats engine's resume with a fresh acceptance and decides the round as a fresh round does.
A live fixer beside a reviewer that is not live is returned to `BurlerProducer`, which stops the fixer and runs a fresh round, unchanged; `BurlerProducer`'s lone-live-reviewer branch stays, unreached by this engine.

`contracts/stencils/stencils.go`'s `roleOpeningStencils` keys the two orchestrators by `burler-advisor-1` and `burler-chair` in place of `burler-review` and `burler-fix`.
Each touched file's header comment (`engine.go`, `handoff.go`, `resume.go`, `prompt.go`) is rewritten to the seat wiring, and `repairReportBeforeGate`'s doc names `roundTable` as where it is composed.

Tests (fakes at the module surface, `PATTERN-test-economy`): `engine_test.go` drives the real seats engine over its package-local scripted fake, now a `seatengine.Shuttle` whose handles also answer `StrandName` (the name the seats engine forms from the geometry's shortname and slug and the spec's role), `Send` (lands every line) and `SessionState`.
- `TestEngine_Run_SpecConstruction`: the chair starts first, then the reviewer; roles `burler-chair` and `burler-advisor-1`; each seat's model, effort, version, skills, review segment, timeout, round and one output; the chair gated by the wrapped `opts.Gate` and not interactive; the reviewer gated by the review-parse entry alone, keeping its pane, with `QuietHold` false.
- `TestEngine_Run_ForkSubagentsSpecWiring`: fork authorization on the reviewer exactly when the profile names a fan, never on the chair.
- `TestEngine_Run_ReviewGateRepairsUnparseableReview`: the review-parse gate re-prompts the reviewer in its own session, and an unparseable review after the budget fails at the strict parse in `accept`.
- `TestEngine_Run_MarkerLifecycle`: the marker exists only after the reviewer's done result passed the cluster audit and the strict parse, and a stale marker is gone before the chair starts.
- `TestEngine_Run_RoundFailureRules`: one row per D4 rule above with the stops each implies (reviewer died or timed out: chair and reviewer stopped, reviewer's outcome, nil error; wait error; audit or strict-parse error; fixer died or timed out while the reviewer runs: reviewer stopped, fixer's outcome; fixer done before release; review changed after the fix; a fake reviewer done with `Gate.Passed` false: the review-gate error and no verdict; success with verdict, findings, gate passthrough, both halves' identities, times and usage), the D5 not-started rows (chair not started starts no reviewer; reviewer not started stops the chair; each with `NotStarted` and the right half's start error), and a seat that cannot be stopped returning `ErrHalfNotStopped` with the way forward.
- `TestEngine_Run_ShuttleOutcomes`, `TestEngine_Run_GateOutcomes` and `TestEngine_Run_ClusterAuditPolicy` keep their rows over the seat wiring.
- `TestEngine_ProbeRound` and `TestEngine_Resume`: probe classification per seat, and resume attaching only with both live, removing a found marker and deciding through the same rules.
  The probe row "found run with its output file on disk stays live" becomes a live reviewer beside a gone fixer that the probe stops, reported `HalfDone` by its file with no handle, and a probe row whose reviewer stop fails wants `ErrHalfNotStopped`.
  The resume row "a round without both halves live is refused and nothing is stopped" wants the reviewer stopped by the probe and the refusal, and a new row calls `Resume` with a hand-built both-live `LiveRound` and wants the absent-table refusal with nothing started or stopped.
- `prompt_test.go` and `template_test.go` call the re-signed `composePrompt`; the orchestrator assertions (among them the fixer's wait line) read the seat prompts `seatengine.Prompts` composes from `roundTable`, and `TestComposePrompt_ParentDirective` asserts the chair names the told parent (or the no-parent variant), the reviewer names the chair, and no instruction file carries either.
- `burlercli/wiring_test.go`: `diedShuttle` becomes a `seatengine.Shuttle` double on a geometry with a shortname, its handles naming their strands as the seats engine expects; `TestRunCmd` keys the halves by the new roles.

Docs: `internal/burlerengine/doc.go` describes the seat table, the acceptance hook, the failure rules over the seat result, the probe and resume, and the `ErrHalfNotStopped` wrap; `docs/overview.md`'s burler entry, module table row (builds on seats) and the "Following one spawn down the stack" steps name the seat table in place of two shuttle runs.

**ImpactSummary:** Rewires burlerengine's round onto seatengine and re-signs New, whose wiring and smoke-test callers move to seatengine.RunnerShuttle; BurlerProducer's seam, LiveRound's exported shape and ErrHalfNotStopped are kept.
