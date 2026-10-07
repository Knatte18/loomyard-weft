# burler-cost — decision record

## Goal

Cut the cost and wall time of a Burler round by splitting it into two agents that start together: a cheap reviewer that writes the review only, and a stronger fixer that orients itself while the review runs, then validates and fixes the findings.
Let `loom.yaml` pick each half's model per round, so round 1, round 2 and later rounds can run different base models.

## Scope

In:

- `internal/burlerengine`: one round runs two shuttle runs, a reviewer and a fixer, started concurrently and joined by a Go-mediated handoff; the engine is told a strand remover, and the prior-rounds hydration prose names disputed findings beside deferred ones.
- A non-waiting live-run probe on the shuttle attach seam, the one shuttle addition.
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
Go then waits on both runs concurrently; the first failure of either half decides the round (D6), and otherwise the round completes when the fixer's run does, which D6 allows only after the marker was written.
Shed still sees one `BurlerRound` step with one outcome.
The existing `Start`/`StartGated` and `Wait` already let a caller hold two runs at once, but nothing stops a run that is blocked in `Wait`.
`burlerengine` is therefore told a strand remover, the told-seam shape `websterengine` uses for `RemoveStrandIfLive`, and removing a half's strand is how D6 stops it.
The round returns the failing half's outcome, never the stopped half's consequential died outcome or error.

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
The fixer never writes the review file in any fix scope: its rendered write surface lists the fixer-report only, so the overlay scope's naming of the review path does not reach it.
Its shuttle `OutputFiles` is the fixer-report path alone, and the caller's gate entries (`RunOpts.Gate`, such as Webster-Burler's `verify`) run on the fixer's run only.
`repairReportBeforeGate` names only the fixer-report, since the review is the reviewer's.
The Tier 2 friction directive goes to the fixer only, so one round has one note writer.

Rationale: the fixer is the half that changes the target, so gates over the changed target belong to its session; one friction writer per note path avoids two sessions writing one file.

### D5 — Handoff: a marker file and `lyx burler await-review`

Decision: Go writes a ready marker once the reviewer's run reached done, a strict parse of its review file succeeded and, for a cluster round, its fork audit passed.
The strict parse is a separate condition because the review-parse gate lets a run through once its re-prompt budget is spent, so a passed gate does not prove a parse.
The marker is machine-local, under `.lyx` at the subpath mirroring the round's review file; in standalone geometry, where the review path has no `_lyx` mirror, it sits under `.lyx` in the told state directory at the review file's path relative to the target.
One function derives the marker path from the review path, and both `BurlerProducer` and `lyx burler run` call it and tell the result to the engine with the round's other paths, so any process re-derives the same one and no caller builds its own.
`Engine.Run` removes any existing marker before it spawns either half, so no caller carries that duty, and it returns only after the waiting goroutine of each half it started has returned, so no marker write outlives its attempt.
The fixer's stencil names the marker path and tells it to run `lyx burler await-review <marker-path>` until it reports ready.
The verb is read-only: it polls for that one file and returns as soon as it exists, or after a cap below the agent's default Bash tool timeout, reporting not ready so the agent calls again.
It needs no hub, mode or git repository, prints a JSON envelope (`ready` true or false), and is a cobra subcommand under `lyx burler` per `PATTERN-cli-cobra`.
The steps from the reviewer's done outcome to the marker write (fork audit, strict parse, marker) are one function that the engine's reviewer goroutine and the resume branch (D6) both call.
Bound: the verb reads one caller-named path and writes nothing; lyx code writes the marker, and only after the reviewer's done outcome, fork audit and strict parse for the current attempt, and no marker survives into the next attempt.
Nothing mechanical stops the fixer from releasing itself early: the marker is a plain file and the verb returns ready for any existing path it is handed, so the fixer could create the marker or name another existing file.
What keeps it from either is the write surface and the instruction its prompt states (D4), a prompt-enforced rule named in the Open risks.

Rationale: keeping the fixer inside one turn while it waits avoids a held turn end, which today sends the parent a hold notice; a portable `lyx` verb works on Windows where a shell loop would not; a short cap keeps each call inside the Bash tool's timeout and refreshes the prompt cache between calls.

### D6 — Failure of either half fails the round; resume

Decision: if the reviewer's run ends in anything but done with a strictly parsing review (died, timeout, not started, review still unparseable after the gate's re-prompt budget, fork-audit violation), Go stops the fixer's run and removes its strand, and the round returns that outcome or error as today.
A review still unparseable after the gate's re-prompt budget is the strict-parse error the round returns today, never a gate-failed outcome, since the reviewer's run never reports a failing gate.
Only the fixer's caller gate entries (D4) can end a done run with a failing gate, and `BurlerProducer` maps that to a gate-failed `Stuck` as today.
If the fixer's run dies, times out or never starts while the reviewer runs, Go stops the reviewer's run and removes its strand, and the round returns the fixer's outcome.
If the fixer's run reaches done before the marker was written, it skipped the handoff: Go stops the reviewer's run and removes its strand, and the round returns an error naming the skipped handoff, with Verdict and Findings empty.
Stopping a half is remove-if-live: removing the strand of a half that already finished is a no-op, and a failed removal or a failed liveness probe is returned as the round's error in place of the failing half's outcome.
The round then spawns nothing beside a strand it could not remove, and `BurlerProducer`'s retry does not run.
`BurlerProducer`'s bounded retry and archive of partial round files stay as they are, applied to the round as a whole.
Resume: the live-round probe classifies each half by its own output file without waiting on it: live is an attachable run (a run whose record still reads running is live even when its output file is already on disk, as a reviewer repairing its review in the parse gate is), done is no live run with the output file present, and gone is neither.
The probe result carries each live half's strand guid, and `BurlerProducer` is told the same strand remover as the engine.
With both halves live, it attaches to both with the two waits running concurrently, so the fixer's wait never blocks the reviewer's, removes any marker found first (a marker here is stale, left by an earlier attempt, since a finished reviewer's run is never live), and runs D5's review-ready function when the attached reviewer's run completes; that function returns the review bytes it parsed.
An attached half's outcome is judged by the rules above for a fresh round: an attached fixer that completes before the marker was written, or an attached half that dies or times out while the other is live, stops the other half, removes its strand and ends the round the same way a fresh round ends.
Once the attached fixer is done, D7's comparison runs against the bytes the review-ready function returned.
With exactly one half live, or one half gone and the other done, Go stops the live half and removes its strand, archives the round's outputs and the marker, and spawns both halves again as attempt 1; a review with no fixer-report is an incomplete round today and is re-run the same way.
With neither half live, Go archives the round's outputs and the marker and spawns both halves as attempt 1.
Bound: a round never ends with one half still live, and a half already live is never spawned a second time; the target edits a stopped fixer already made stay in the worktree, and the next attempt's reviewer reviews them.

Rationale: a half-finished round has no usable result, and a live leftover session is the defect class `bugfix-sessions` removed.
A finished reviewer's run is already cleaned up, so a live fixer beside a finished review is the usual resume state; it is restarted rather than attached because D7's comparison needs the bytes parsed in the process that released the fixer, and a resumed process cannot tell whether the live fixer already changed the review file.

### D7 — Result carries both halves

Decision: `burlerengine.Result` keeps one `Outcome`, `Verdict`, `Findings`, `ReviewPath`, `FixerReportPath`, `Gate` and `NotStarted` for the round, and carries the session id, strand guid, last assistant message and run directory of each half separately.
`ForkAudit` and `ClusterWarnings` come from the reviewer's run.
`Gate` is the fixer's gate outcome (D6).
The review file is parsed strictly once, by the reviewer half after its run reached done and before the marker is written.
Verdict and findings are set on the `Result` from that parse only once the fixer's run reached done with its gate passed; a failing fixer gate leaves them empty, as today.
The segment's judge and the bouncer facts read the review file from disk after the fixer ran, so Go keeps the bytes it parsed and, once the fixer's run reached done, compares the review file with them.
A difference makes the round return an error naming the file, with Verdict and Findings empty.
Bound: the comparison covers the review file only; the fixer's rendered write surface (D4) is what keeps it from writing any other round file.

Rationale: callers branch on one outcome, and an operator inspecting a failed round needs both sessions' identities.

### D8 — Fixer may dispute a finding, with evidence

Decision: the fixer fixes every finding at every severity, except a finding whose premise it shows false against the code or fasit.
Such a finding goes into a new `Disputed` section of the fixer-report with the finding id and the evidence (the cited code or fasit text that contradicts it); it is not fixed.
Bound: a dispute is allowed only for a factually wrong premise; severity, size, cost or disagreement with the rubric never justify one, and the existing deferred section keeps its meaning.
No Go code reads the fixer-report's sections, so a `Disputed` section leaves the round's completion unchanged: the report's existence completes it.
The prior-rounds block of the reviewer's prompt names disputed findings beside deferred ones, so the next round's fresh reviewer is told to re-evaluate them and may raise one again.
Nothing else reads a dispute: the segment's judge reads no fixer-report, and convergence still rests on the judge's verdict over a fresh review.
`PATTERN-review-round` changes in the same commit: every finding is fixed, or disputed by the fixer with evidence that its premise is false.

Rationale: the operator asked for a fixer that acts as an extra validator and takes the reviewer's findings with a grain of salt, which the cheaper reviewer makes necessary; requiring evidence and keeping the judge's fresh review as the convergence point stops a dispute from becoming a way to skip work.

### D9 — Models: `review` and `fix` keys, each a model-spec or a per-round list

Decision: `loom.yaml`'s `review` key becomes the reviewer's model-spec and a new `fix` key the fixer's.
Each accepts either one model-spec string (all rounds) or a YAML block list of model-specs, where entry *n* applies to round *n* and the last entry to every later round.
Every entry is parsed at config load, and an empty list is a config error.
`ResolveReview` resolves every entry of both lists at run start, so a bad later-round entry fails before round 1.
Defaults in the template: `review: sonnet[medium]`, `fix: opus[medium]`.
An existing scalar `review` value loads unchanged as an all-rounds reviewer model.
`loomengine.ResolveReview` returns both resolved lists with the round timeout, `shedrecipe.Env` carries them, and `BurlerProducer` picks each half's model, effort and version from its list by the round number it already computes.
`burlerengine.RunOpts` carries the reviewer's and the fixer's model and effort separately.
`review_timeout_min` stays the deadline of each half's run, counted from that half's own spawn: the fixer's waiting for the review counts against its deadline, and the reviewer's own deadline bounds that wait.

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

- `PATTERN-told-geometry`: `burlerengine` derives no path; the marker path is told with the round's other paths, derived from the review path by one function outside `burlerengine`, never built from the per-attempt instruction directory.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the marker sits under `.lyx` at the mirrored subpath of the review file, and in standalone geometry under `.lyx` in the told state directory at the review file's path relative to the target, reached through `lyxdirs`.
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
- The fixer's orientation while waiting is worth its tokens; the operator chose the overlap for wall time.

## Open risks

- Both halves' read-only discipline before the handoff is prompt-enforced, as A-before-B is today; a fixer that edits before the marker breaks review independence, and no mechanical check catches it.
- The fixer's write surface is prompt-enforced: only the review file is compared after the fixer (D7), and a fixer that creates the marker itself, names another existing file to the wait verb, or writes another round file is caught by review alone.
- The fixer's polling costs one small API call per wait cap; a reviewer that runs long multiplies them.
- A fixer that disputes freely could stall convergence; D8's evidence rule and the judge's fresh review bound it, but only review catches misuse.
- Two concurrent sessions per round raise peak load on a host already running several runs.
- Changing `--model`/`--effort` on `lyx burler run` breaks any script that passes them; only in-repo callers and tests are known.

## Acceptance criteria

- A round starts the reviewer and the fixer before either finishes, under strands `…:burler-review` and `…:burler-fix`.
- The fixer's wait verb reports not ready until Go writes the marker, and Go writes it only after the reviewer's run reached done with a strictly parsing review (and, for a cluster round, a passing fork audit); a marker left or written late by an earlier attempt never releases the fixer of a later one.
- A standalone `lyx burler run` derives its marker path with the same function as a hub round, and a marker left by an earlier run of the same review path never releases its fixer.
- `lyx burler await-review` returns ready at once when the marker exists, returns not ready after its cap when it does not, writes nothing, and runs outside a hub.
- A reviewer that dies, times out or leaves its review unparseable after the review-parse gate's re-prompt budget ends the round with the fixer's strand removed, and the unparseable review ends it with the strict-parse error and no verdict; a fixer that dies while the reviewer runs ends the round with the reviewer's strand removed, without waiting for the reviewer to finish its review; a fixer that reaches done before the marker was written ends the round with the reviewer's strand removed and an error naming the skipped handoff; the round returns the failing half's outcome, never the stopped half's consequential one; a strand that cannot be removed ends the round with an error and no retry or respawn; no round ends with a live half.
- A fixer that changes the review file after the handoff ends the round with an error naming the file and no verdict.
- The review-parse gate re-prompts the reviewer's session; a caller gate entry such as `verify` re-prompts the fixer's session and names only the fixer-report for repair.
- A fixer-report with a `Disputed` section completes the round exactly as one without, and the fixer stencil names the section, its evidence requirement and its factual-premise bound; the next round's prior-rounds block names disputed findings beside deferred ones.
- The fixer's rendered write surface never lists the review file, in any fix scope.
- Each half's spec carries the round timeout, only the fixer's prompt carries the friction directive, and both halves' specs carry the skill list and the PATTERN directive.
- `review` and `fix` each accept a scalar or a list; round 1 takes entry 1, round *n* past the list takes the last entry, the two halves resolve independently, an empty list or an unparseable entry fails config load, and an existing scalar `review` loads unchanged.
- A `BurlerRound` recipe row with `model` or `effort` is rejected.
- `lyx burler run` accepts the four per-half model flags and rejects `--model`.
- A resumed run whose round has both halves live reattaches to both with the waits concurrent, removes any marker it finds, and never spawns a second agent for either; with one half live or one done and one gone, it finds the live half without waiting on it, stops it and re-runs the round from attempt 1, and with neither half live it re-runs from attempt 1.
- On a resumed round, an attached fixer that completes before the marker was written, or an attached half that dies or times out while the other is live, ends the round as a fresh round does, and a fixer that changes the review file after a resumed handoff ends it with an error naming the file and no verdict.
- `Result` exposes each half's session id, strand guid, last message and run directory.

## Notes for the plan writer

- `burlerengine/engine.go` builds one `shuttleengine.Spec` and calls `RunGated`; the split needs the `StartGated`/`Wait` form, which `websterengine` already uses.
- The live-round probe in `shedadapters/burler.go` (`probeLiveRound`) matches on `OutputFiles`; with two runs, each half is probed through the non-waiting probe by its own single output file.
- `shedadapters/burler.go` builds its own gate spec with `ReviewGateEntry` on the attach path; it must follow D3/D4's split.
- `contracts/recipes/loom-recipe.yaml` comments mention row-level model keys and "single reviewer"; they follow D10.
- `_lyx/config/loom.yaml` in a worktree is not this task's to edit; only the template and its reconcile behavior change.
