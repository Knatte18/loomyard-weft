# burler-cost — decision record

## Goal

Cut the cost and wall time of a Burler round by splitting it into two agents that start together: a cheap reviewer that writes the review only, and a stronger fixer that orients itself while the review runs, then validates and fixes the findings.
Let `loom.yaml` pick each half's model per round, so round 1, round 2 and later rounds can run different base models.

## Scope

In:

- `internal/burlerengine`: one round runs two shuttle runs, a reviewer and a fixer, started concurrently and joined by a Go-mediated handoff.
- Round stencils under `contracts/stencils/burler/`: a reviewer orchestrator and a fixer orchestrator replace the single-agent round orchestrator; the step files are reassigned to the half that runs them.
- A new `lyx burler` verb the fixer runs to wait for the review.
- Model selection: `loom.yaml` gains a `fix` key beside `review`; both accept one model-spec or a per-round list.
- Threading the two model lists through `loomengine`, `shedrecipe.Env`, `entries_burler.go` and `shedadapters.BurlerProducer`, which picks each half's model by round.
- `shedadapters.BurlerProducer`'s live-round probe and retry, adapted to two strands.
- `lyx burler run` flags for the two halves' models.
- Docs in the same commit: `internal/burlerengine/doc.go`, `internal/shedadapters/doc.go` where it describes the round, the `loom.yaml` template comments, `PATTERN.md` and `pattern/PATTERN-review-round.md`.

Out:

- A generic two-agent producer in shuttle (`two-agent-producer`) and agent-to-agent messaging (`creel`).
- Turning on a cluster fan for any recipe row; each row keeps today's fan setting.
- Per-fork models (Claude Code runs forks under the session's model).
- Bouncer and judge model selection (`judge` key) and cache TTL (`cache-ttl-per-role`).
- Prompt-cache TTL tuning for the waiting fixer.

## Decisions

### D1 — One round, two shuttle runs inside `burlerengine`

Decision: `burlerengine.Engine.Run` starts a reviewer run and a fixer run through the shuttle seam's start-then-wait form, both before waiting on either, and returns one `Result` for the round.
Shed still sees one `BurlerRound` step with one outcome.
No shuttle capability is added: the existing `Start`/`StartGated` and `Wait` already let a caller hold two runs at once, as Webster's master does.

Rationale: the operator placed a two-agent producer inside one producer, never as two shed producers (`two-agent-producer`), and this round needs only a one-way handoff, not a dialogue, so it needs neither creel nor a generic shuttle capability.
`burlerengine` stays a told-geometry producer and its caller chain (caller → burler → shuttle) is unchanged.

### D2 — Roles and strand names

Decision: the reviewer runs under role `burler-review` and the fixer under `burler-fix`, both declared as constants in `burlerengine`, replacing role `burler`.
Strand names follow `PATTERN-agent-name`: `<shortname>:<slug>:burler-review` and `<shortname>:<slug>:burler-fix`.

Rationale: two live strands in one round need distinct names, and `PATTERN-agent-name` makes the spawning module own its role constants.

### D3 — Reviewer half

Decision: the reviewer runs today's job A unchanged in substance: explore (step 1), review against the fasit with the profile's cluster fan if it names one (step 2), and write the review file.
Its shuttle `OutputFiles` is the review path alone, its gate spec is the review-parse entry (`ReviewGateEntry`) alone, and fork subagents are authorized exactly when the profile names a fan.
Its write surface is the review file only, in every fix scope, and it runs no mutating git command.
Its stencil states that it never edits a target file and never fixes.

Rationale: the reviewer never grades its own fix, and the review-parse gate belongs to the session that wrote the review.

### D4 — Fixer half

Decision: the fixer starts with the same fasit, target and PATTERN directive as the reviewer and orients in the target (step 1).
It then calls the wait verb (D5) until the review is ready, reads it, validates each finding against the code, and runs today's job B (step 3) with the profile's fix scope, writing the fixer-report.
Until the wait verb reports the review ready, the fixer edits no file and runs no mutating git command; its stencil states this as the blocking sequencing rule that today's orchestrator states for A-before-B.
Its shuttle `OutputFiles` is the fixer-report path alone, and the caller's gate entries (`RunOpts.Gate`, such as Webster-Burler's `verify`) run on the fixer's run only.
`repairReportBeforeGate` names only the fixer-report, since the review is the reviewer's.
The Tier 2 friction directive goes to the fixer only, so one round has one note writer.

Rationale: the fixer is the half that changes the target, so gates over the changed target belong to its session; one friction writer per note path avoids two sessions writing one file.

### D5 — Handoff: a marker file and `lyx burler await-review`

Decision: Go writes a ready marker into the round's machine-local instruction directory under `.lyx/burler/round-*/` once the reviewer's run reached done, its review-parse gate passed and, for a cluster round, its fork audit passed.
The fixer's stencil names the marker path and tells it to run `lyx burler await-review <marker-path>` until it reports ready.
The verb is read-only: it polls for that one file and returns as soon as it exists, or after a cap below the agent's default Bash tool timeout, reporting not ready so the agent calls again.
It needs no hub, mode or git repository, prints a JSON envelope (`ready` true or false), and is a cobra subcommand under `lyx burler` per `PATTERN-cli-cobra`.
Bound: the verb reads one caller-named path and writes nothing; it cannot release the fixer early, because only Go writes the marker, and only after D3's checks.

Rationale: keeping the fixer inside one turn while it waits avoids a held turn end, which today sends the parent a hold notice; a portable `lyx` verb works on Windows where a shell loop would not; a short cap keeps each call inside the Bash tool's timeout and refreshes the prompt cache between calls.

### D6 — Failure of either half fails the round

Decision: if the reviewer's run ends in anything but done with a passing review gate (died, timeout, not started, gate exhausted, fork-audit violation), Go stops the fixer's run and removes its strand, and the round returns that outcome or error as today.
If the fixer's run dies, times out or never starts while the reviewer runs, Go stops the reviewer's run and removes its strand, and the round returns the fixer's outcome.
`BurlerProducer`'s bounded retry and archive of partial round files stay as they are, applied to the round as a whole.
Bound: a round never ends with one half still live.

Rationale: a half-finished round has no usable result, and a live leftover session is the defect class `bugfix-sessions` removed.

### D7 — Result carries both halves

Decision: `burlerengine.Result` keeps one `Outcome`, `Verdict`, `Findings`, `ReviewPath`, `FixerReportPath`, `Gate` and `NotStarted` for the round, and carries the session id, strand guid, last assistant message and run directory of each half separately.
`ForkAudit` and `ClusterWarnings` come from the reviewer's run.
Verdict and findings are still parsed from the review file only after both runs reached done and the fixer's gate passed, exactly as today.

Rationale: callers branch on one outcome, and an operator inspecting a failed round needs both sessions' identities.

### D8 — Fixer may dispute a finding, with evidence

Decision: the fixer fixes every finding at every severity, except a finding whose premise it shows false against the code or fasit.
Such a finding goes into a new `Disputed` section of the fixer-report with the finding id and the evidence (the cited code or fasit text that contradicts it); it is not fixed.
Bound: a dispute is allowed only for a factually wrong premise; severity, size, cost or disagreement with the rubric never justify one, and the existing deferred section keeps its meaning.
A disputed finding reaches the next round through the prior fixer-report hydration, and the next round's fresh reviewer may raise it again; convergence still rests on the judge's verdict over a fresh review.
`PATTERN-review-round` changes in the same commit: every finding is fixed, or disputed by the fixer with evidence that its premise is false.

Rationale: the operator asked for a fixer that acts as an extra validator and takes the reviewer's findings with a grain of salt, which the cheaper reviewer makes necessary; requiring evidence and keeping the judge's fresh review as the convergence point stops a dispute from becoming a way to skip work.

### D9 — Models: `review` and `fix` keys, each a model-spec or a per-round list

Decision: `loom.yaml`'s `review` key becomes the reviewer's model-spec and a new `fix` key the fixer's.
Each accepts either one model-spec string (all rounds) or a YAML block list of model-specs, where entry *n* applies to round *n* and the last entry to every later round.
Every entry is parsed and resolved at config load, and an empty list is a config error.
Defaults in the template: `review: sonnet[medium]`, `fix: opus[medium]`.
An existing scalar `review` value loads unchanged as an all-rounds reviewer model.
`loomengine.ResolveReview` returns both resolved lists with the round timeout, `shedrecipe.Env` carries them, and `BurlerProducer` picks each half's model, effort and version from its list by the round number it already computes.
`burlerengine.RunOpts` carries the reviewer's and the fixer's model and effort separately.
`review_timeout_min` stays the deadline of each half's run.

Rationale: the operator measured the reviewer as good enough on `sonnet[medium]` and the fixer as the half that must be stronger; a per-round list lets a fresh base model review after round 1 converged, and a scalar keeps today's configs valid without a migration.

### D10 — Recipe row `model`/`effort` keys are removed

Decision: the `BurlerRound` row loses its `model` and `effort` config keys; `configRejectUnknown` then rejects them.
`timeout_s` stays.

Rationale: no recipe row sets them, and after the split a single row-level model would be ambiguous between the two halves; model choice lives in `loom.yaml` only.

### D11 — `lyx burler run` flags

Decision: `lyx burler run`'s `--model` and `--effort` are replaced by `--review-model`, `--review-effort`, `--fix-model` and `--fix-effort`.

Rationale: the standalone round runs both halves and needs both models; reusing `--model` for one half would be ambiguous.

### D12 — Skills and PATTERN directive

Decision: both halves load the skills `burlerSkills` names today and both receive the PATTERN directive.

Rationale: the reviewer judges against the same rules the fixer applies.

## Constraints

- `PATTERN-told-geometry`: `burlerengine` derives no path; the marker path is built from the round directory it already creates under `Geometry.AnchorPath`.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the marker sits under `.lyx`, reached through `lyxdirs`.
- `PATTERN-cli-cobra`, `PATTERN-sandbox-coverage`: the new verb is a cobra subcommand with `Short` and JSON errors, and the sandbox suite exercises it or excludes it with a reason.
- `PATTERN-agent-name`: role names are `burlerengine` constants.
- `PATTERN-review-round`: changed by D8, in the same commit, with its background file.
- `PATTERN-fabric-git`: unchanged; the reviewer runs no mutating git, the fixer commits only under `FixScopeSource`.
- `PATTERN-stencil-ownership`: the new stencils are read through `stencilstore` from the told stencils directory.
- `PATTERN-role-skills-typed`: skills are named on each launch spec, never asked for in a stencil.
- `PATTERN-parent-directive`: each new top-level stencil renders the parent directive.
- `PATTERN-producer-pointer-rule`: the fixer stencil points at the review format in the reviewer's step file rather than restating it.
- `PATTERN-shuttle-provider-seam`: no provider specifics leave `claudeengine`.
- `PATTERN-config-strictness`: `loom.yaml` loading keeps its strict path; a malformed `fix` or `review` list fails load.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: new behavior is covered at `burlerengine`'s and `shedadapters`' public surfaces with the fake shuttle; real-engine smoke tests stay in their tier.
- `PATTERN-comment-line-breaks`: every new or changed Go comment uses semantic line breaks.
- Docs land in the same commit (CLAUDE.md): `burlerengine/doc.go` is rewritten for the two-agent round.

## Auto-mode assumptions

- The handoff is one-way, reviewer to fixer; the fixer never asks the reviewer anything.
- "B then forks its lens fan as today" means the reviewer keeps the cluster flow wherever a profile names a fan; no recipe row names one today, and enabling one is out of scope.
- The round number used to pick a model is the segment-local round `BurlerProducer` already computes from its run directory.
- The default `fix: opus[medium]` matches the operator's stopgap in `bugfix-fabric`; `review: sonnet[medium]` matches the operator's 2026-10-07 statement.
- The fixer's orientation while waiting is worth its tokens; the operator chose the overlap for wall time.
- A disputed finding is acceptable to the operator's "grain of salt" only with evidence (D8).

## Open risks

- Both halves' read-only discipline before the handoff is prompt-enforced, as A-before-B is today; a fixer that edits before the marker breaks review independence, and no mechanical check catches it.
- The fixer's polling costs one small API call per wait cap; a reviewer that runs long multiplies them.
- A fixer that disputes freely could stall convergence; D8's evidence rule and the judge's fresh review bound it, but only review catches misuse.
- Two concurrent sessions per round raise peak load on a host already running several runs.
- Changing `--model`/`--effort` on `lyx burler run` breaks any script that passes them; only in-repo callers and tests are known.

## Acceptance criteria

- A round starts the reviewer and the fixer before either finishes, under strands `…:burler-review` and `…:burler-fix`.
- The fixer's wait verb reports not ready until Go writes the marker, and Go writes it only after the reviewer's run reached done with a parsing review (and, for a cluster round, a passing fork audit).
- `lyx burler await-review` returns ready at once when the marker exists, returns not ready after its cap when it does not, writes nothing, and runs outside a hub.
- A reviewer that dies, times out or exhausts its review-parse gate ends the round with the fixer's strand removed; a fixer that dies while the reviewer runs ends the round with the reviewer's strand removed; no round ends with a live half.
- The review-parse gate re-prompts the reviewer's session; a caller gate entry such as `verify` re-prompts the fixer's session and names only the fixer-report for repair.
- The fixer-report's `Disputed` section is accepted by the round, and a disputed finding is not counted as unfixed by anything that checks the report.
- `review` and `fix` each accept a scalar or a list; round 1 takes entry 1, round *n* past the list takes the last entry, the two halves resolve independently, an empty list or an unparseable entry fails config load, and an existing scalar `review` loads unchanged.
- A `BurlerRound` recipe row with `model` or `effort` is rejected.
- `lyx burler run` accepts the four per-half model flags and rejects `--model`.
- A resumed run whose round has a live half reattaches to it and never spawns a second agent for the same half.
- `Result` exposes each half's session id, strand guid, last message and run directory.

## Notes for the plan writer

- `burlerengine/engine.go` builds one `shuttleengine.Spec` and calls `RunGated`; the split needs the `StartGated`/`Wait` form, which `websterengine` already uses.
- The live-round probe in `shedadapters/burler.go` (`probeLiveRound`) matches on `OutputFiles`; with two runs, each half's probe matches its own single output file.
- `shedadapters/burler.go` builds its own gate spec with `ReviewGateEntry` on the attach path; it must follow D3/D4's split.
- `contracts/recipes/loom-recipe.yaml` comments mention row-level model keys and "single reviewer"; they follow D10.
- `_lyx/config/loom.yaml` in a worktree is not this task's to edit; only the template and its reconcile behavior change.
