# Weaver: a cold-start implementer beside Webster, with model escalation

## Goal

Add Weaver, a second implementer for tapestry's implementer row beside Webster.
Weaver runs each batch of the approved plan as a fresh cold agent on its own model, escalates the model when an attempt fails, and has no long-lived master.
Webster stays fork-based and keeps its behaviour.

## Scope

In:

- The `weaver` implementer value, its engine package, its `weaver.yaml` config module, its stencils and its run directory.
- Making the loom paths that read Webster's run state today read the active implementer's (D13).
- The `contracts/specs/implementer-spec.md` updates D1 names.

Out:

- Parallel batches, a worktree per batch, merge-back and a DAG batchifier (operator, 2026-11-15).
- A long-lived Master Weaver session (operator, 2026-11-15).
- Pushing from inside the row (see Open risks).

## Decisions

### D1 — Weaver is the `weaver` value of tapestry's implementer row

`internal/shedrecipe`'s closed `implementers` table gains `weaver`, built by a Weaver constructor that takes the row's other config keys, `gates` included, as Webster's does.
A run chooses Weaver with `tapestry.implementer: weaver`, hub-wide or per task through `run_settings`; the default stays `webster`.
Weaver builds the same row with the same edges, inputs and outcomes as `contracts/specs/implementer-spec.md` pins.
That spec gains: `weaver` in its table; an Out line saying an implementer may commit its run directory and the plan directory during the run; and a Stuck route that says the stop goes to shed's stop chain (rule table, tackler, arbiter, parent), as the driver does today.
Guard: the row refuses when the run's other implementer has begun batches in the current generation, naming two ways forward: set `tapestry.implementer` back, or reset the other implementer's run directory.
A run never switches implementer with work half built.
Rationale: the implementer seam exists for this choice, so no recipe row, recipe file or routing edge changes.

### D2 — One row drives every batch in Go; its stops go through shed's stop chain

The Weaver row's producer runs the loop itself: form the next batch, run its attempts, check and record it, repeat, then the plan-level verify (D10).
No LLM runs on the happy path besides the batch agents.
The row returns Stuck with a reroute on a plan conflict (D6), and otherwise only on the stops D5, D6 and D10 name or on a fault in Weaver's own config or state.
A batch stop's reason names the batch and its cards, and a verify stop's reason names the findings; both name the plan directory, Weaver's state file, the last report, the journal and the decision record.
Bound: the row adds no edge and no verb; the arbiter's per-run spawn cap and the driver's repair cap bound how often a stop re-enters the row (D7).
Rationale: the operator decided Weaver reuses Shed so the tackler and arbiter handle a stuck batch, and that a Go happy path is wise (2026-11-15).

### D3 — Batches are formed one at a time from the remaining cards

Before each batch, Weaver reads the plan through `internal/planparser`, drops the cards its state records done (matched by card slug), and partitions the rest with the active batcher profile through `internal/batcher`.
It runs the first batch of that partition and discards the rest.
A batch's card set and start commit are recorded in state when the batch begins and stay fixed for every attempt of that batch.
Rationale: the board's "batches formed as the run goes", run sequentially; a rename a batch reports rewrites the pending cards (D11), and the next partition sees the rewritten plan.

### D4 — A batch attempt is one cold strand in the task worktree

Each attempt starts a fresh shuttle strand in the task worktree, in a reed pane with no operator, with the attempt's rung model (D5) as its `ModelChoice`.
Its prompt, rendered from a Weaver stencil read through `stencilstore`, names only what the agent reads: its batch's cards, the decision record, the journal (D8), and on a retry the previous attempt's failure digest and uncommitted paths, as Webster's recovery prompt does.
The agent commits its work on the task branch and writes its report as the strand's output file.
The strand carries the batch's card gates and the row's `gates` entries as must-pass shuttle gates with their re-prompts, so the agent fixes its own gate failures before it ends: Builder's one-shot self-resolve.
Rationale: Weaver is Webster's recovery strand used as the normal path (board, Shape).

### D5 — The escalation ladder is a list of model specs in `weaver.yaml`

`weaver.yaml` holds `ladder`, an ordered list of model specs, default `sonnet[medium]`, `opus[medium]`, `opus[high]`.
A batch's first attempt runs on the first rung; a failed attempt (D6) starts the next attempt on the next rung.
A failed attempt on the last rung stops the row (D2).
The verify fixer (D10) climbs the same ladder.
Each spec resolves through `modelspec` and reaches the spawn only as a `shuttleengine.ModelChoice` from `shuttleengine.NewModelChoice`.
An empty ladder or a spec that does not resolve refuses when the row builds, naming `lym config weaver` as the way forward.
Bound: before its first stop a batch gets one model-failed attempt per rung, plus the same-rung retries D6 allows; after a stop, D7's one attempt per re-entry, under the arbiter's spawn cap.
Rationale: the board's escalation ladder per failed attempt, set in config; one list serves batches and fixer alike.

### D6 — What fails an attempt, what retries on the same rung, and what stops at once

An attempt fails, and climbs the ladder, when:

- its strand times out or ends with no report while shuttle reports it finished;
- its report says `FAILED`;
- its gates still fail after the strand's re-prompts;
- it leaves the tree unclean, or the post-batch checks (D11) fail.

A strand lost to the machinery with no report (shuttle's respawn-eligible or API-error verdicts, a reed loss, a reboot) gets one fresh attempt on the same rung; a second loss counts as a failed attempt.
A report saying `NEEDS_PARENT` stops the row at once with the report's `need`, without climbing.
A plan-conflict refusal at the batch's begin takes the row's reroute.

### D7 — Re-entering the row after a stop

When the row is stepped again after a batch stop, Weaver runs the full post-batch checks (D11) on the stopped batch against its recorded start commit, with the card gates at HEAD.
If they pass and the tree is clean, the batch is recorded done with no spawn, since the arbiter or a human fixed it.
If not, the batch gets one more attempt on the last rung; failing that stops the row again.
An arbiter fix carries no report, so it carries no renames; a pending card that names a glyph the fix removed is caught by the post-batch drift check and refuses at that card's begin, as in Webster.
After a verify stop, re-entry runs the verify again, and a failure gets one fixer on the last rung before the row stops again.
Rationale: the arbiter, a one-shot fixer that commits and then names a goto target, needs the row to judge its fix by the run's own checks.

### D8 — A run journal carries memory to later batches and the arbiter

Each batch report carries a `journal` field: what the batch struggled with and what later batches must keep in mind.
Go appends it, under a heading naming the batch and its attempt, to `journal.md` in Weaver's run directory; failed attempts' entries are kept.
Every later attempt's prompt and every fixer's prompt names the journal for the agent to read first.
The arbiter's description of a tapestry run names the active implementer's run directory; for Weaver it names the plan directory, the state file, the reports and the journal, so a stop on any row reaches them.
A report with an empty `journal` is accepted.
Rationale: the operator's trial of a journal in place of a long-lived master (2026-11-15), and the board's call that Weaver's description names where the plan, reports and deviations lie.

### D9 — Weaver's run directory and its commits

Weaver owns `_lym/weaver/`: its state file, its reports directory, `journal.md` and `summary.md`; prompts and locks go under `.lym/weaver/`.
Both are named through `lymdirs`.
The state file follows `internal/state` and refuses no unknown key.
After each batch ends, done or failed, Go commits `_lym/weaver/` together with the plan directory, through the same records-commit path Webster's run commit uses, so plan rewrites and the journal never leave the tree unclean.
Weaver never pushes; the landing rows do.
A PR rework round archives `_lym/weaver/` into the prior generation as it does `_lym/webster/`, so the next generation starts fresh.
Rationale: the journal must outlive a crash (2026-11-15), and a commit per batch puts it on the branch.

### D10 — The plan-level verify, its fixer and the run record

After the last batch, Weaver runs the plan-level verify through `verifytree.Verify`, in held gate slots, with the gate key `weaver.yaml`'s `verify_gate` names (default `webster`) and one rerun for flakiness, as Webster's gate does.
A failure starts a cold fixer strand on the ladder (D5) with the findings and the journal, carrying the row's `gates` entries as D4's strands do; the fixer commits `fix: <summary>`, and the verify runs again.
A failure after the last rung stops the row; a pass returns Done.
Go writes `summary.md` in the shared final-summary format: the batches and their outcomes, the flaky identities as `## Integration suite triage`, and each fixer's findings and commits as `## Verify gate fixes`, so Describe carries them into the change description as it does Webster's.
Rationale: the implementer contract's Done means the plan-level verify passed; the `webster` gate key spares every hub a new `gate.yaml` entry.

### D11 — Webster's batch machinery is shared, not copied

Weaver reuses Webster's report contract and parser, plan fingerprint and plan-conflict refusal, post-batch checks (done checks, handle binding, scope guard, renames rewriting pending cards), suspect-path checks, failure digest, card gate rendering, strand await and verify-failure record.
Where a piece is private to `internal/websterengine`, it moves to a package both implementers import, and Webster's behaviour stays as it is.
Weaver's report adds `journal` to the shared contract; Webster's forks may leave it empty.
Fork transcript audits stay Webster's alone.
Rationale: PATTERN-shed-driver's shared-machinery rule; two copies of the post-batch checks would drift.

### D12 — Crash and resume

The row's producer holds a run lock for its whole call.
State records the batch in flight, its attempt and rung before the strand is spawned, and the strand once it is.
On a new call Weaver first records a finished report that was not yet recorded; then it attaches to a live run of the in-flight attempt, found through `shuttleengine.FindRun` even when state never recorded its strand; only then does it treat a run that is gone with no report per D6.
Rationale: a row is re-stepped after any crash or move, and a gated strand relaunched after a crash sits idle with its report already written; spawning twice would race two agents on one tree.

### D13 — Loom's readers of implementer state read the active implementer's

The loom paths that read Webster's run state today read the active implementer's through one seam, so each sees Weaver's state when Weaver runs:

- the plan gate's done-card freeze (`internal/loomshed/gates.go`) and the Plan-Review rubric's frozen-card note (`internal/loomshed/publishfailure.go`);
- the renumber verbs' bound at the last begun card (`internal/loomcli/renumber.go`);
- the plan-conflict record and its route claim, which move to one implementer-neutral path both implementers write, named through `lymdirs`, and which the tapestry recipe's Plan-Review text names instead of `.lym/webster/plan-conflict.md`;
- the decision-notice guard (`internal/loomcli/decision.go`), which treats Weaver's batch and fixer strand roles as implementer strands, so a halted run's leftover strand is not woken and a live one gets the inline decision.

Rationale: a reroute to Plan-Burler-Review after Weaver has built cards must freeze them and see the conflict, or the plan is rewritten under landed work.

## Constraints

- PATTERN-shed-recipe-registry: the `Implementer` entry's closed table is the one exception to the registry map; Weaver joins that table and adds no registry entry.
- PATTERN-shuttle-provider-seam: rung specs reach a spawn only as `ModelChoice` built by `NewModelChoice`; strands stop only through shuttle's stop verb; a negative finish consults the output files first.
- PATTERN-reed-panes: every batch and fixer strand is a reed session started through shuttle, never headless.
- PATTERN-fabric-boundary: the per-batch commit of `_lym/weaver/` and the plan directory goes through fabric, as Webster's run commit does.
- PATTERN-config-layers: `weaver.yaml` is a new module loaded through `configengine` and registered in `configreg`; its template ships the defaults.
- PATTERN-lym-files: `_lym/weaver/`, `.lym/weaver/` and the shared plan-conflict path are named only by `lymdirs`; locks through `lock`.
- PATTERN-tolerant-records: Weaver's state file never refuses an unknown key.
- PATTERN-gate-slot: the plan-level verify runs in held gate slots through `verifytree.Verify`.
- PATTERN-stencil-ownership: Weaver's prompts are stencils read at call time through `stencilstore`, rendering the parent and edit directives.
- PATTERN-refusal-way-forward: every Weaver refusal and Stuck reason names its way forward, and a test reaches it.
- A Codex spec may stand on a rung only as far as `codex-same-as-claude` has given Codex sessions what Claude sessions in the same role get; the default ladder is Claude-only.

## Auto-mode assumptions

- The loop over batches lives in the row's producer (D2).
- Batches run sequentially in the task worktree (D4).
- The default ladder is `sonnet[medium]`, `opus[medium]`, `opus[high]` (D5).
- A machinery loss retries once on the same rung (D6).
- A re-entered stop gets one attempt on the last rung (D7).
- The journal is a report field Go appends (D8).
- The verify reuses the `webster` gate key by default (D10).

## Open risks

- The operator's 2026-11-15 decision says "each batch is a producer". This record reads it as one producer that runs every batch, whose stops reach shed's chain (D2), not a row or a nested shed run per batch. Question for the parent: is that reading right?
- The board says the journal is pushed with the run, so it outlives a move to another machine. Weaver commits after every batch (D9) but, per the implementer contract, never pushes, and `push-before-act-takeover` pushes at shed transitions, of which the whole Weaver row is one. Question for the parent: should Weaver push after each batch as an exception to "the implementer never pushes", should `push-before-act-takeover` push at points inside a producer, or is a push at the row's end enough?

## Acceptance criteria

- With `tapestry.implementer: weaver`, the tapestry recipe builds the Webster row with Weaver, the row accepts its `gates` key, and its Done, reroute and Stuck routes match Webster's (D1).
- The row refuses with its way forward when the other implementer has begun batches in the current generation (D1).
- A three-card plan the batcher splits in two runs two batch strands in order, each on the first rung, and the row returns Done after the verify passes, with no other LLM spawned (D2, D4, D10).
- After a batch that reports a rename, the next batch is partitioned from the rewritten pending cards; done cards are matched by slug; a batch's card set stays fixed across its attempts (D3, D11).
- A batch prompt names the batch's cards, the decision record and the journal; a retry's prompt adds the failure digest and uncommitted paths; the strand carries the card gates and the row's gates (D4, D8).
- A failed attempt starts the next attempt on the next rung's model; a failure on the last rung returns Stuck whose reason names the batch, the state file, the report and the journal (D2, D5, D6).
- A `FAILED` report, a failing gate and an unclean tree each climb the ladder; a machinery loss retries once on the same rung; a `NEEDS_PARENT` report returns Stuck at once with its need (D6).
- An empty ladder or an unresolvable spec refuses at row build, naming `lym config weaver` (D5).
- Re-entering after a batch stop whose batch passes the post-batch checks records it done with no spawn; one that fails gets one last-rung attempt; re-entering after a verify stop reruns the verify (D7).
- Each batch's journal entry, failed attempts' included, is appended to the journal; `_lym/weaver/` and the plan directory are committed after each batch; a PR rework round archives `_lym/weaver/` (D8, D9).
- The arbiter's description of a run with Weaver active names Weaver's run directory and journal (D8).
- A verify failure starts a fixer on the ladder and reruns the verify; a failure after the last rung returns Stuck; `summary.md` carries the triage and fixes sections Describe reads (D10).
- Webster's existing tests pass unchanged after the shared machinery moves (D11).
- A call after a crash records an unrecorded finished report, attaches to a live in-flight run even when state never recorded its strand, and never spawns a second agent for one attempt (D12).
- With Weaver active, the plan gate freezes Weaver's done cards, the renumber verbs stop at Weaver's last begun card, a plan conflict reaches Plan-Burler-Review, and a decision notice is held back from a halted run's Weaver strand (D13).
- Review: `contracts/specs/implementer-spec.md` lists `weaver`, allows commits during the run and names the stop chain; the tapestry Plan-Review text names the shared plan-conflict path; Weaver's package doc states its loop, ladder, journal and run directory (D1, D2, D5, D8, D9, D13).

## Notes for the plan writer

- `internal/websterengine`'s `recoverSpawn`, `RecoverAwait` and `RenderRecoveryPrompt` are the closest model for a batch attempt; `postBatchChecks` and `checkRecoveredSuspects` hold the checks D11 shares.
