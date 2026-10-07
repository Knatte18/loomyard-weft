# Card 7 — Split the round into a concurrent reviewer and fixer

**Create:**
- `plan:internal/burlerengine#Handle` -> `type Handle interface`
- `plan:internal/burlerengine#RunnerShuttle` -> `func RunnerShuttle(r *shuttleengine.Runner) Shuttle`
- `plan:internal/burlerengine#ErrHalfNotStopped` -> `var ErrHalfNotStopped error`
- `plan:internal/burlerengine#Engine.join` -> `func (e *Engine) join(p *Profile, opts RunOpts, review, fix Handle) (Result, error)`
- `internal/burlerengine/shuttle.go`
- `internal/burlerengine/shuttle_test.go`
- `internal/burlerengine/handoff.go`

**Edit:**
- `internal/burlerengine/engine.go`
- `internal/burlerengine/profile.go`
- `internal/burlerengine/prompt.go`
- `internal/burlerengine/doc.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/profile_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `.gitattributes`
- `internal/burlercli/cli.go`
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
- `docs/shared-libs/stencil.md`
- `pattern/PATTERN-review-round.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `contracts/stencils/burler/burler-template-round-orchestrator.md`

**Uses:**
- `plan:internal/burlermarker#Path`
- `plan:internal/burlerengine#StrandRemover`
- `plan:internal/burlerengine#NewReedStrandRemover`
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

What the handoff and the moved gates let through, and their bound: the marker is the one edge that releases the fixer, and the engine cannot see a fixer that creates the marker itself, edits the target before it, names another existing file to the wait verb, or writes a round file other than its fixer-report;
the review-parse gate no longer follows the fixer's session, and the single-session A-before-B ordering becomes a prompt-enforced rule on a separate session.
What bounds them: lyx code alone writes the marker, only after the current attempt's reviewer reached done, passed the fork audit (cluster rounds) and parsed strictly, and `Run` removes a stale marker before either half starts;
the fixer's rendered write surface never lists the review file and its stencil states the sequencing rule (card 5);
after the fixer is done the engine compares the review file with the bytes it parsed, which covers the review file only;
a fixer that reaches done before the marker, and a half that cannot be stopped (`ErrHalfNotStopped`, never retried or archived over), end the round with an error.
The remainder is prompt-enforced and caught by review alone, as the decision record's Open risks name.

Seams (`shuttle.go`).
`Handle` is the started run the engine waits on: `StrandGUID() string`, `RunDir() string`, `Wait() (shuttleengine.Result, error)`, which `*shuttleengine.Run` satisfies.
`Shuttle` becomes `StartGated(shuttleengine.Spec, shuttleengine.GateSpec) (Handle, error)` in place of `Run`/`RunGated`;
`RunnerShuttle` adapts a `*shuttleengine.Runner` to it.
`New(shuttle Shuttle, remover StrandRemover, geom Geometry, cfg Config, stencilsDir, frictionDir string)`, with `StrandRemover` from card 6.

Profile and options (`profile.go`).
`Profile.ReadyMarkerPath` is told with the round's other paths, resolved by `validate` like them, and required non-empty.
`RunOpts.Model`/`Effort` become `RunOpts.Review` and `RunOpts.Fix`, each a `ModelChoice` mapped onto its half's `Spec.Model`, `Effort` and `Version`.

Roles and specs (`engine.go`).
Constants `burler-review` and `burler-fix` replace role `burler` (D2), so strands are `<shortname>:<slug>:burler-review` and `…:burler-fix`.
Both specs carry `opts.Timeout`, `opts.Round` and `burlerSkills`, and both prompts carry the PATTERN directive (D12).
Reviewer: `OutputFiles` is the review path alone, gate spec is `ReviewGateEntry(reviewPath)` alone, `ForkSubagents` exactly when the profile names a fan (D3).
Fixer: `OutputFiles` is the fixer-report path alone, gate spec is `opts.Gate` with every entry wrapped by `repairReportBeforeGate`, which now names the fixer-report only (D4).
One helper builds both specs and both gate specs, so card 8's resume probes with the same ones.

Prompts (`prompt.go`).
`composePrompt` renders the two orchestrator stencils from card 5 and four instruction files into the per-round directory: explore for the reviewer (no friction directive), explore for the fixer (the friction directive, the round's only note writer), review (step 2, reviewer only) and fix (step 3, fixer only).
The fixer orchestrator's `review_format_path` is the rendered step-2 file and its `ready_marker_path` is `Profile.ReadyMarkerPath`.
`fixScopeRules`' overlay write surface names the target paths plus the fixer-report only, so the fixer's rendered write surface never lists the review file in any fix scope.
`priorRoundsBlock` tells the reviewer to re-evaluate the prior rounds' deferred and disputed items (D8).
`clusterRulesBlock` no longer tells the reviewer the review must be on disk "before job B touches anything" (the reviewer has no job B; it says the review is complete when written; `prompt_test.go` asserts the old sentence and moves to the new one), and `toolUseRules` speaks of the explore and review steps instead of "job A", since both halves read it.
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
That error's message ends with `way forward: run "lyx reed remove <guid>", then re-step the row`, naming the stuck half's guid.
It is the refusal the loom section's row `producer re-run: superseded strand not removed` already holds for a row that cannot remove a live strand, so `contracts/specs/refusal-spec.md` widens that row's trigger in this commit to a BurlerRound row whose half stays live, and the `ErrHalfNotStopped` test row asserts the clause (`PATTERN-refusal-way-forward`).
The round returns the failing half's outcome, never the stopped half's consequential died outcome or error, and `join` returns only after both waiting goroutines have returned, so no marker write outlives its attempt and no half is left live.
The skipped-handoff and changed-review errors, and the failures to remove a stale marker or to create and write the marker, are burler-module errors that `contracts/specs/refusal-spec.md`'s Out of scope paragraph keeps out of its table, as the strict-parse and cluster-audit errors already are, so they land no row;
their messages name the skipped handoff, the changed file, and the marker path respectively.

`Run`: validate, remove any existing marker (no caller carries that duty), compose, start the reviewer then the fixer through `StartGated`, then `join`.
A start failing with `ErrNotStarted` is that half's `OutcomeDied` with `NotStarted` set and a nil error, as `RunGated` reported it;
but `StartGated` returns only `(nil, err)` there and drops the died-or-timeout class and the session, strand and run dir of the torn-down run that `RunGated`'s Result carried, so such a half reports `OutcomeDied` whatever the class, empty identity fields, and its half struct's `StartError` carries the start error's text, which names the run dir, the strand and whether the strand was removed;
a reviewer that fails to start returns that result with the fixer never started, and a fixer that fails to start stops the already-started reviewer first and returns the fixer's result.
Any other error from `StartGated` is a pre-strand failure (spec validation, such as a fixer-report path that already exists, run-directory creation, `AddStrand`, run-state persistence) and is not an `OutcomeDied` result:
a plain error from the reviewer's `StartGated` is returned, wrapped as `burler: shuttle run: %w` as today, with the fixer never started;
a plain error from the fixer's `StartGated` stops the already-started reviewer through the remover first and is then returned the same way, or is returned wrapped as `ErrHalfNotStopped` in its place when that stop fails, so no start failure leaves a half live.
`Result` keeps one `Outcome`, `Verdict`, `Findings`, `ReviewPath`, `FixerReportPath`, `Gate` and `NotStarted`, takes `ForkAudit` and `ClusterWarnings` from the reviewer, and replaces `SessionID`, `StrandGUID`, `LastAssistantMessage` and `RunDir` with `Review` and `Fix` fields, each a struct carrying those four for its half plus `StartError`, set only for a half that never started.

Callers.
`burlercli/run.go`: `--model`/`--effort` are replaced by `--review-model`, `--review-effort`, `--fix-model`, `--fix-effort` (D11), so `--model` is rejected as an unknown flag;
`run` sets `Profile.ReadyMarkerPath` with `burlermarker.Path`, root and base taken from the wired geometry (hub: the anchor for both; standalone: the target and the state directory);
`burlerCLI` in `cli.go` gains `markerRoot` and `markerBase` fields for them, which `wireHub` and `wireStandalone` set beside `mode` and `stateDir`, since `burlerengine.Engine` keeps its geometry unexported;
the envelope reports a `review` and a `fix` object with each half's identity, plus `readyMarkerPath`, in place of the top-level identity keys;
`burlermarker.Path` refuses a `review-path` outside the root, so a standalone `run` now refuses a profile whose `review-path` is outside the target, with that error and its way forward (place the review path under the target);
the `Long` help describes the two halves and the four flags and states that constraint.
`burlercli/wiring.go` and `loomcli/wiring.go` build `RunnerShuttle(runner)` and `NewReedStrandRemover(reedEngine)` (card 6) and pass both to `New`.
`shedadapters/burler.go`: each round sets `ReadyMarkerPath` from `burlermarker.Path(anchor, anchor, reviewPath)`, with `BurlerDeps.AnchorPath` added and checked absolute in `NewBurlerProducer`;
each attempt sets `RunOpts.Review` and `RunOpts.Fix` from `Models.Pick(round)`;
a runner error wrapping `ErrHalfNotStopped` returns without archiving and without the retry, as the attach-error path does, since a live agent may still write the round's files;
messages that named the round's session and run dir name both halves', and the double-failure message for a half that never started names its `StartError` in place of the identity it does not have.
The existing pair-file attach probe stays until card 8 replaces it;
neither half declares both output files, so the retained probe matches no run and is inert in the meantime.
`shedrecipe/entries_burler.go` passes `env.AnchorPath` (required absolute) into `BurlerDeps`.
`shedfake` follows the `Result` field change.

Docs in this commit: `burlerengine/doc.go` is rewritten for the two-agent round (halves, roles, handoff, failure rules, result);
`shedadapters/doc.go`'s round description names two strands;
`docs/overview.md`'s burler entry says two agents, reviewer then fixer, and its execution-stack walkthrough has `burler` start two shuttle runs and the fixer wait for the reviewer's review in place of one handler agent writing both files;
`docs/shared-libs/stencil.md`'s `burler` bullet names the two orchestrators and four instruction files in place of one thin orchestrator and three;
`pattern/PATTERN-review-round.md` says the reviewer and fixer are separate sessions and the fixer touches nothing before Go's marker.

Tests, at `burlerengine`'s public surface with a fake `Shuttle` whose handles block until the test releases them and a fake remover that ends a removed handle's `Wait` as died and can be told to fail, in `engine_test.go`:
the existing spec-construction, shuttle-outcome and gate-outcome tests are rewritten for two halves, covering that both halves start before either finishes, under the two roles, with single-file `OutputFiles`, their gate specs, the round timeout, the skill list, and the friction directive in the fixer's explore file only, and `Result`'s per-half identities;
two new table-driven functions cover the rest, one per behaviour.
The first is the marker lifecycle: the marker is absent until the reviewer is done with a parsing review (and, for a cluster round, a passing audit), and a stale marker is removed before either half starts.
The second is the round's failure rules, one row each: every failure row of `join` above, asserting the round's outcome or error, the removed guid and that no fake handle is left blocked;
the `ErrHalfNotStopped` row, asserting the way-forward clause;
a reviewer that never started, asserting `OutcomeDied`, `NotStarted`, empty identity, the reviewer's `StartError` text and that the fixer was never started, and a fixer that never started, asserting the same for the fixer plus the removed reviewer guid;
a reviewer whose `StartGated` fails with a plain error, asserting the error is returned and the fixer was never started, and a fixer whose `StartGated` fails with a plain error, asserting the reviewer's guid was removed and the error is returned, with the stop failing for a second row that asserts the `ErrHalfNotStopped` wrap;
a changed review after the handoff; a fixer-report carrying a `Disputed` section completing like any other.
`shuttle_test.go` is one new table-driven function over the real adapter, `RunnerShuttle(shuttleengine.NewRunner(…))` built over `shuttlefake.Reed` and `shuttlefake.Engine` as `internal/websterengine/recoverbatch_test.go` builds its runner: a started run returns a `Handle` naming its strand, and a failed start (`shuttlefake.Reed`'s `AddErr`) returns an interface-nil `Handle` with the error, so no typed-nil handle reaches `join`.
The three new functions raise the `internal/burlerengine` row of `cmd/lyx/testdata/test-budget.yaml` by three; every other test in this card is a row, or a rewritten assertion, of an existing function (`TestTestBudget_PackagesStayWithinBudget` fails on a package over its row).
`prompt_test.go` asserts the fixer's rendered write surface never lists the review path in either fix scope, both orchestrators' markers render, and the prior-rounds block names disputed findings.
`profile_test.go` adds the `ReadyMarkerPath` resolution and empty-value rows.
`template_test.go` replaces its single `orchestrator` row and `orchestratorMarkerValues` with a row and a values map for each new orchestrator, so the fill and missing-required-marker sweep covers both.
`burlercli` tests assert the four flags, `--model` rejected, and `readyMarkerPath` equal to `burlermarker.Path` over the wired root and base.
`shedadapters` tests assert the per-round marker path, both halves' model picks, and the non-archiving `ErrHalfNotStopped` exit, as rows of existing tests.
`burlercli`'s tests also assert a standalone `run` refuses a `review-path` outside the target with the way forward, as a row.
Grep for every reworded message and every assertion on the removed `Result` fields and envelope keys, and update each.

**ImpactSummary:** burlerengine's seam, `New`, `RunOpts`, `Profile` and `Result` change shape; every caller (burlercli, loomcli, shedadapters, shedrecipe, shedfake) moves with them in this card, together with their docs, the refusal-spec row and the budget row, and the deleted orchestrator stencil is referenced only by its embed and registry entry in `stencils.go`, the `burler` role entry, its claims in `claims_test.go`, its `.gitattributes` line, `prompt.go` and the `burlerengine` doc, all targets of this card, and by the step files' header comments, which card 5 rewrites.
