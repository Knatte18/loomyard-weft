# Card 6 — Split the round into a concurrent reviewer and fixer

**Create:**
- `plan:internal/burlerengine#Handle` -> `type Handle interface`
- `plan:internal/burlerengine#StrandRemover` -> `type StrandRemover interface`
- `plan:internal/burlerengine#RunnerShuttle` -> `func RunnerShuttle(r *shuttleengine.Runner) Shuttle`
- `plan:internal/burlerengine#ErrHalfNotStopped` -> `var ErrHalfNotStopped error`
- `plan:internal/burlerengine#Engine.join` -> `func (e *Engine) join(p *Profile, opts RunOpts, review, fix Handle) (Result, error)`
- `internal/burlerengine/shuttle.go`
- `internal/burlerengine/handoff.go`
- `internal/burlerengine/remover.go`

**Edit:**
- `internal/burlerengine/engine.go`
- `internal/burlerengine/profile.go`
- `internal/burlerengine/prompt.go`
- `internal/burlerengine/doc.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/burlerengine/profile_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `.gitattributes`
- `internal/burlercli/run.go`
- `internal/burlercli/wiring.go`
- `internal/burlercli/cli_test.go`
- `internal/burlercli/wiring_test.go`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomcli/wiring.go`
- `internal/testkit/shedfake/shedfake.go`
- `docs/overview.md`
- `pattern/PATTERN-review-round.md`

**Delete:**
- `contracts/stencils/burler/burler-template-round-orchestrator.md`

**Uses:**
- `plan:internal/burlermarker#Path`
- `plan:internal/burlerengine#ModelChoice`
- `plan:internal/burlerengine#RoundModels`
- `plan:internal/burlerengine#RoundModels.Pick`
- `internal/shuttleengine#Runner.StartGated`
- `internal/shuttleengine#ErrNotStarted`
- `internal/websterengine#removeStrandIfLive`
- `internal/burlerengine#ReviewGateEntry`
- `internal/burlerengine#auditClusterRound`
- `internal/burlerengine#ParseReview`

**Intent:**
Implement D1–D5, D7, D9's per-half models, D11 and D12: one `Engine.Run` call starts a reviewer run and a fixer run, joins them through a Go-written marker, and returns one `Result`.
Shed still sees one `BurlerRound` step with one outcome.

Seams (`shuttle.go`, `remover.go`).
`Handle` is the started run the engine waits on: `StrandGUID() string`, `RunDir() string`, `Wait() (shuttleengine.Result, error)`, which `*shuttleengine.Run` satisfies.
`Shuttle` becomes `StartGated(shuttleengine.Spec, shuttleengine.GateSpec) (Handle, error)` in place of `Run`/`RunGated`;
`RunnerShuttle` adapts a `*shuttleengine.Runner` to it.
`StrandRemover` is `RemoveStrandIfLive(guid string) error`, the told-seam shape of `websterengine`'s `removeStrandIfLive`: a no-op for a strand reed does not report live, an error for a failed liveness probe or a failed removal, and a Warn log for every live strand it removes (`PATTERN-spawn-observability`).
`remover.go` adds the reed-backed implementation over `shuttleengine.ReedOps`.
`New(shuttle Shuttle, remover StrandRemover, geom Geometry, cfg Config, stencilsDir, frictionDir string)`.

Profile and options (`profile.go`).
`Profile.ReadyMarkerPath` is told with the round's other paths, resolved by `validate` like them, and required non-empty.
`RunOpts.Model`/`Effort` become `RunOpts.Review` and `RunOpts.Fix`, each a `ModelChoice` mapped onto its half's `Spec.Model`, `Effort` and `Version`.

Roles and specs (`engine.go`).
Constants `burler-review` and `burler-fix` replace role `burler` (D2), so strands are `<shortname>:<slug>:burler-review` and `…:burler-fix`.
Both specs carry `opts.Timeout`, `opts.Round` and `burlerSkills`, and both prompts carry the PATTERN directive (D12).
Reviewer: `OutputFiles` is the review path alone, gate spec is `ReviewGateEntry(reviewPath)` alone, `ForkSubagents` exactly when the profile names a fan (D3).
Fixer: `OutputFiles` is the fixer-report path alone, gate spec is `opts.Gate` with every entry wrapped by `repairReportBeforeGate`, which now names the fixer-report only (D4).
One helper builds both specs and both gate specs, so card 7's resume probes with the same ones.

Prompts (`prompt.go`).
`composePrompt` renders the two orchestrator stencils from card 5 and four instruction files into the per-round directory: explore for the reviewer (no friction directive), explore for the fixer (the friction directive, the round's only note writer), review (step 2, reviewer only) and fix (step 3, fixer only).
The fixer orchestrator's `review_format_path` is the rendered step-2 file and its `ready_marker_path` is `Profile.ReadyMarkerPath`.
`fixScopeRules`' overlay write surface names the target paths plus the fixer-report only, so the fixer's rendered write surface never lists the review file in any fix scope.
`priorRoundsBlock` tells the reviewer to re-evaluate the prior rounds' deferred and disputed items (D8).
Delete `burler-template-round-orchestrator.md`, its embed, registry entry, `burler` role entry in `roleOpeningStencils`, its claims and its `.gitattributes` line.

Handoff (`handoff.go`).
`reviewReady` is the one function from the reviewer's done outcome to the marker (D5): for a cluster round copy the `ForkAudit` and run `auditClusterRound`, then strictly parse the review file with `ParseReview`, then write the marker (creating its parent directory), returning the parsed bytes, verdict, findings and cluster warnings.
`join` waits on both handles in two goroutines and decides the round:
- The reviewer's run ends in anything but done, or `reviewReady` fails: stop the fixer and return the reviewer's outcome (died, timeout) with a nil error, or the audit or strict-parse error (an unparseable review after the gate's budget is the strict-parse error, never a gate-failed outcome).
- The fixer's run dies or times out while the reviewer runs: stop the reviewer without waiting for its review and return the fixer's outcome.
- The fixer's run reaches done before the marker was written: stop the reviewer and return an error naming the skipped handoff, Verdict and Findings empty.
  Whether the marker was written is the reviewer goroutine's recorded fact, set when the marker write returns, never the order two results arrive in.
- Otherwise the round completes with the fixer's run: a failing fixer gate returns the fixer's `Gate` with Verdict and Findings empty, as today;
  a done fixer with a passing or absent gate compares the review file with the bytes `reviewReady` parsed, and a difference returns an error naming the file with Verdict and Findings empty (D7);
  only then are Verdict and Findings set from that parse.
Stopping a half is `RemoveStrandIfLive` on its guid; a removal or liveness-probe failure returns an error wrapping `ErrHalfNotStopped` in place of the failing half's outcome.
The round returns the failing half's outcome, never the stopped half's consequential died outcome or error, and `join` returns only after both waiting goroutines have returned, so no marker write outlives its attempt and no half is left live.

`Run`: validate, remove any existing marker (no caller carries that duty), compose, start the reviewer then the fixer through `StartGated`, then `join`.
A start failing with `ErrNotStarted` is that half's `OutcomeDied` with `NotStarted` set and a nil error, as `RunGated` reported it;
a fixer that fails to start stops the already-started reviewer first.
`Result` keeps one `Outcome`, `Verdict`, `Findings`, `ReviewPath`, `FixerReportPath`, `Gate` and `NotStarted`, takes `ForkAudit` and `ClusterWarnings` from the reviewer, and replaces `SessionID`, `StrandGUID`, `LastAssistantMessage` and `RunDir` with `Review` and `Fix` fields, each a struct carrying those four for its half.

Callers.
`burlercli/run.go`: `--model`/`--effort` are replaced by `--review-model`, `--review-effort`, `--fix-model`, `--fix-effort` (D11), so `--model` is rejected as an unknown flag;
`run` sets `Profile.ReadyMarkerPath` with `burlermarker.Path`, root and base taken from the wired geometry (hub: the anchor for both; standalone: the target and the state directory);
the envelope reports a `review` and a `fix` object with each half's identity, plus `readyMarkerPath`, in place of the top-level identity keys;
the `Long` help describes the two halves and the four flags.
`burlercli/wiring.go` and `loomcli/wiring.go` build `RunnerShuttle(runner)` and the reed-backed remover and pass both to `New`.
`shedadapters/burler.go`: each round sets `ReadyMarkerPath` from `burlermarker.Path(anchor, anchor, reviewPath)`, with `BurlerDeps.AnchorPath` added and checked absolute in `NewBurlerProducer`;
each attempt sets `RunOpts.Review` and `RunOpts.Fix` from `Models.Pick(round)`;
a runner error wrapping `ErrHalfNotStopped` returns without archiving and without the retry, as the attach-error path does, since a live agent may still write the round's files;
messages that named the round's session and run dir name both halves'.
The existing pair-file attach probe stays until card 7 replaces it.
`shedrecipe/entries_burler.go` passes `env.AnchorPath` (required absolute) into `BurlerDeps`.
`shedfake` follows the `Result` field change.

Docs in this commit: `burlerengine/doc.go` is rewritten for the two-agent round (halves, roles, handoff, failure rules, result);
`shedadapters/doc.go`'s round description names two strands;
`docs/overview.md`'s burler entry says two agents, reviewer then fixer;
`pattern/PATTERN-review-round.md` says the reviewer and fixer are separate sessions and the fixer touches nothing before Go's marker.

Tests, at `burlerengine`'s public surface with a fake `Shuttle` whose handles block until the test releases them and a fake remover that ends a removed handle's `Wait` as died and can be told to fail, one table-driven test per behaviour in `engine_test.go`:
both halves start before either finishes, under the two roles, with single-file `OutputFiles`, their gate specs, the round timeout, the skill list, and the friction directive in the fixer's explore file only;
the marker is absent until the reviewer is done with a parsing review (and, for a cluster round, a passing audit), and a stale marker is removed before either half starts;
each failure row of `join` above, asserting the round's outcome or error, the removed guid, that no fake handle is left blocked, and the `ErrHalfNotStopped` row;
a changed review after the handoff; a fixer-report carrying a `Disputed` section completing like any other;
`Result`'s per-half identities.
`prompt_test.go` asserts the fixer's rendered write surface never lists the review path in either fix scope, both orchestrators' markers render, and the prior-rounds block names disputed findings.
`profile_test.go` adds the `ReadyMarkerPath` resolution and empty-value rows.
`burlercli` tests assert the four flags, `--model` rejected, and `readyMarkerPath` equal to `burlermarker.Path` over the wired root and base.
`shedadapters` tests assert the per-round marker path, both halves' model picks, and the non-archiving `ErrHalfNotStopped` exit.
Grep for every reworded message and every assertion on the removed `Result` fields and envelope keys, and update each.

**ImpactSummary:** burlerengine's seam, `New`, `RunOpts`, `Profile` and `Result` change shape; every caller (burlercli, loomcli, shedadapters, shedrecipe, shedfake) moves with them in this card.
