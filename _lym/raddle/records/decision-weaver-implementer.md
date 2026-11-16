## Goal

Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam

Takes: internal/runplan as implementer-shared-run-plan delivers it: the run-plan state and batch records Weaver's state file embeds, the report contract with journal, the post-batch checks, suspect checks, digest, card gate rendering, strand await and verify-failure record, each driven by the geometry and way-forward text Weaver passes
Delivers: the weaver value of tapestry's implementer row: its engine package, the weaver.yaml config module, its stencils, _lym/weaver/ and .lym/weaver/ named through lymdirs, the lym weaver reset verb, the active-implementer seam every loom reader of implementer state goes through, the implementer-neutral plan-conflict path, and the implementer-spec updates
Exports: internal/weaverengine with its row producer and reset; internal/weavercli with lym weaver reset; the active-implementer seam the D1 guard, plan gate, renumber verbs, decision-notice guard, Describe, the PR rework archive and merge-in read

## Scope

The part changes these packages:

- internal/weaverengine (about 5000 changed lines)
- internal/weavercli (about 800 changed lines)
- internal/runplan (about 120 changed lines)
- internal/websterengine (about 120 changed lines)
- internal/shedrecipe (about 120 changed lines)
- internal/loomshed (about 300 changed lines)
- internal/loomcli (about 350 changed lines)
- internal/shedadapters (about 100 changed lines)
- internal/landingshed (about 30 changed lines)
- internal/fabriccli (about 50 changed lines)
- internal/configreg (about 10 changed lines)
- internal/lymdirs (about 40 changed lines)
- internal/hubgeom (about 60 changed lines)
- internal/standalonegeom (about 60 changed lines)
- cmd/lym (about 10 changed lines)
- contracts/stencils/weaver (about 250 changed lines)
- contracts/specs (about 30 changed lines)
- contracts/recipes (about 10 changed lines)

## Decisions

### D1 — Weaver is the `weaver` value of tapestry's implementer row

`internal/shedrecipe`'s closed `implementers` table gains `weaver`, built by a Weaver constructor that takes the row's other config keys, `gates` included, as Webster's does.
A run chooses Weaver with `tapestry.implementer: weaver`, hub-wide or per task through `run_settings`; the default stays `webster`.
Weaver builds the same row with the same edges, inputs and outcomes as `contracts/specs/implementer-spec.md` pins.
That spec gains: `weaver` in its table; an Out line saying an implementer may commit its run directory and the plan directory during the run; and a Stuck route that says the stop goes to shed's stop chain (rule table, tackler, arbiter, parent), as the driver does today.
Guard: the row refuses when the run's other implementer has begun batches in the current generation.
It reads both implementers' state through D13's seam, so a missing state file counts as no begun batches.
A corrupt one refuses, naming the file and, as the way forward, repairing it; a corrupt state file never counts as missing.
For Weaver's state file the repair may restore its last version from the per-batch weft commits (D9) only when the task branch's HEAD is the end commit that version records for its last batch, so no attempt has committed since,
and `lym reed status` lists no live strand of Weaver's batch or fixer roles, so no attempt is in flight;
otherwise the way forward is `lym weaver reset` (D2).
A restored state has no begin record for the batch in progress, so the next begin would capture HEAD as the batch start,
and the post-batch checks would take their delta from a start that excludes the earlier attempts' commits.
The begun-batches refusal names two ways forward: set `tapestry.implementer` back, or reset the other implementer's run and the work it built, with `lym webster reset --to start` for Webster's and `lym weaver reset` (D2) for Weaver's.
A test reaches each.
A run never switches implementer with work half built.
Rationale: the implementer seam exists for this choice, so no recipe row, recipe file or routing edge changes.

### D2 — One row drives every batch in Go; its stops go through shed's stop chain

The Weaver row's producer runs the loop itself: form the next batch, run its attempts, check and record it, repeat, then the plan-level verify (D10).
No LLM runs on the happy path besides the batch agents.
The row returns Stuck with a reroute on a plan conflict (D6), and otherwise only on the stops D5, D6 and D10 name or on a fault in Weaver's own config or state.
A batch stop's reason names the batch and its cards, and a verify stop's reason names the findings; both name the plan directory, Weaver's state file, the last report, the journal and the decision record.
Bound: the row adds no edge.
It adds one verb, `lym weaver reset`, which moves the code branch back to the run's start commit, the first batch's recorded start commit, as `lym webster reset --to start` does.
It restores the plan directory in the weft to its content at the run's start, the weft commit the first begin records (D3).
That reverts every plan edit made since, a Plan-Review's included, since the run starts over from the plan it started with.
It archives the run record in place, as Webster's reset renames its state file and reports directory inside its own run directory:
Weaver's state file, reports directory, `journal.md`, `summary.md` and `outcome.yaml` are renamed under UTC-stamped names inside `_lym/weaver/`, named through `lymdirs`.
No reader of the run record reads those names, Describe's prior-generation readers included, so a later reset or a PR rework archive carries them inert.
It clears `.lym/weaver/` and commits the plan restore and the archived run record together through fabric; Webster's run directory is untouched.
Over a corrupt state file it reads the run's start commit and plan commit from the state file's last committed version (D9).
That version names no attempt of the batch in progress, so the reset then also refuses while `lym reed status` lists a live strand of Weaver's batch or fixer roles, naming `lym reed remove <guid>` as the way forward.
With no version committed it refuses, naming as its way forward removing the corrupt state file,
and moving the task branch and the weft's plan directory back by hand, with git, to where each stood before the run's first commit on it.
The verb refuses while Weaver's run lock is held, which only a live producer does since the lock is a `flock` the kernel drops when its holder exits, naming waiting for the row to stop or `lym loom pause --after` the implementer row as its way forward.
Before it moves, archives or clears anything, it probes for the strand of every batch attempt and fixer state records, whatever its recorded outcome,
by the strand's recorded output file through shuttle's `ProbeGated`, since state may never have recorded the strand's guid (D12).
It refuses while a probe finds a run, naming two ways forward: `lym weaver reset --stop-strand`, which stops every strand the probes find through the probe's handle (`Stop`) and then resets,
or re-stepping the row so that it attaches (D12), under `lym loom pause --after` the implementer row when the run should stop once the row ends.
A probe error refuses, naming the error and shuttle's way forward.
Without `--stop-strand` the reset stops no strand of a run a probe finds.
A probe that finds none has already removed, as `ProbeGated` does, the strand of an earlier run whose record is terminal or whose agent shuttle judges gone;
the strand of an attempt whose outcome is recorded but which was not yet stopped is one of them.
A stop re-enters the row through the tackler's resume, under `tackler_max_spawns`, through the arbiter's goto, under `arbiter_max_spawns`, and through a parent re-step, which is unbounded by design (D7).
One producer that loops over the batches in Go, each batch stop returned as the row's Stuck so it reaches shed's stop chain, is the reading of the operator's decision that each batch is a producer.
Rationale: the operator decided Weaver reuses Shed so the tackler and arbiter handle a stuck batch, and that a Go happy path is wise (2026-11-15).

### D3 — Batches are formed one at a time from the remaining cards

Before each batch, Weaver reads the plan through `internal/planparser`, drops the cards its state records done (matched by card slug), and partitions the rest with the active batcher profile through `internal/batcher`.
At each batch begin it records the full partition in state: the batches already done, kept as recorded, then the fresh partition of the remaining cards.
Each batch's number is its first card's number, as the shared batch identity gives it.
It runs the first new batch; the later new batches stay recorded as pending until the next begin replaces them.
Pending batches live only in the recorded partition and get no batch record until they begin.
The first begin also records the weft commit that holds the plan, for `lym weaver reset` (D2).
The shared post-batch checks read this recorded partition (D11), so the later-reference check sees the pending cards and only the run's true last batch reads as last.
A batch's card set and start commit are recorded in state when the batch begins and stay fixed for every attempt of that batch.
State records the begin under Weaver's state-mutation lease (D13).
Rationale: the board's "batches formed as the run goes", run sequentially; a rename a batch reports rewrites the pending cards (D11), and the next partition sees the rewritten plan.

### D4 — A batch attempt is one cold strand in the task worktree

Each attempt starts a fresh shuttle strand in the task worktree, in a reed pane with no operator, with the attempt's rung model (D5) as its `ModelChoice`.
Its prompt, rendered from a Weaver stencil read through `stencilstore`, names only what the agent reads: its batch's cards, the decision record, the journal (D8), and on a retry the previous attempt's failure digest and uncommitted paths, as Webster's recovery prompt does.
The agent commits its work on the task branch and writes its report as the strand's output file.
The strand carries the batch's card gates and the row's `gates` entries as must-pass shuttle gates with their re-prompts, so the agent fixes its own gate failures before it ends: Builder's one-shot self-resolve.
Rationale: Weaver is Webster's recovery strand used as the normal path (board, Shape).

### D5 — The escalation ladder is a list of model specs in `weaver.yaml`

`weaver.yaml` holds `ladder`, an ordered list of model specs, default `sonnet[medium]`, `opus[medium]`, `opus[high]`, and `fixer_timeout_min` and `verify_gate` (D10).
A batch's first attempt runs on the first rung; a failed attempt (D6) starts the next attempt on the next rung.
A failed attempt on the last rung stops the row (D2).
The verify fixer (D10) climbs the same ladder.
Each spec resolves through `modelspec` and reaches the spawn only as a `shuttleengine.ModelChoice` from `shuttleengine.NewModelChoice`.
An empty ladder or a spec that does not resolve refuses when the row builds, naming `lym config weaver` as the way forward.
Bound: before its first stop a batch gets one model-failed attempt per rung, each with at most one same-rung retry after a machinery loss (D6).
After a ladder stop, it gets D7's one last-rung attempt per re-entry; after a `NEEDS_PARENT` stop, it resumes the ladder at the rung that asked (D7).
Re-entries are bounded by the caps D2 names.
Rationale: the board's escalation ladder per failed attempt, set in config; one list serves batches and fixer alike.

### D6 — What fails an attempt, what retries on the same rung, and what stops at once

An attempt fails, and climbs the ladder, when:

- its strand times out or ends with no report while shuttle reports it finished;
- its report says `FAILED`;
- its gates still fail after the strand's re-prompts;
- it leaves the tree unclean, or the post-batch checks (D11) fail.

A strand lost to the machinery with no report (shuttle's respawn-eligible or API-error verdicts, a reed loss, a reboot) gets one fresh attempt on the same rung; a second loss counts as a failed attempt.
That retry applies to each attempt, D7's re-entry attempt and D10's fixer strands included.
Every attempt that ends without being recorded done, D10's fixer strands included, has its strand stopped before the next attempt starts or the row returns, whatever ended it: a failure, a timeout, a machinery loss, a `NEEDS_PARENT` report or a fault in Weaver's own config or state.
For a batch attempt, state first records the attempt's outcome and failure cause (D9), and then the strand is stopped through shuttle's stop verb.
Weaver awaits a fixer until `gatefixer.Await` returns a terminal result, which stops the fixer's strand, and then records the fixer's outcome.
When that state write fails, the strand is stopped all the same and the row returns Stuck naming the fault;
re-entry then attaches through D12; a report on disk is judged through the attach's result, and only an attempt with no report counts as a machinery loss.
When the stop fails, or `gatefixer.Await` warns that a fixer's stop failed, no next attempt starts:
the row returns Stuck naming the strand's guid and, as its way forward, `lym reed remove <guid>`, with the attempt's outcome already recorded, so D12's first step stops it again on re-entry.
The arbiter then meets a Stuck whose reason names the live strand; D12's first step meets a failed stop the same way.
A report saying `NEEDS_PARENT` stops the row at once with the report's `need`, without climbing.
A plan-conflict refusal at the batch's begin takes the row's reroute.

### D7 — Re-entering the row after a stop

When the row is stepped again after a batch stop, Weaver first re-reads the plan.
When every card the recorded partition holds is still a card of the plan, and every plan card is in a recorded batch, it restamps Weaver's plan baseline and maps the recorded partition onto the current plan (D11) before it runs any check.
So an edit to a card's text, the stopped batch's own included, carries into the next attempt.
Otherwise re-entry refuses, naming as ways forward putting the plan's card set back as the partition records it, or `lym weaver reset` to start the run over from the plan it started with (D2).
Re-entry after a `NEEDS_PARENT` stop then starts the next attempt on the rung that asked, with the ladder above it still open, and never takes the no-spawn path below.
After any other batch stop, Weaver runs the full post-batch checks (D11) on the stopped batch against its recorded start commit, with the card gates at HEAD.
If they pass and the tree is clean, the batch is recorded done with no spawn, since the arbiter or a human fixed it.
Bound: since a row `gates` entry needs a strand to run in, a row with `gates` entries never takes the no-spawn path; its stopped batch gets the last-rung attempt, whose strand runs them over the fix.
Otherwise the batch gets one more attempt on the last rung; failing that stops the row again.
An arbiter fix carries no report, so it carries no renames; a pending card that names a glyph the fix removed is caught by the post-batch drift check and refuses at that card's begin, as in Webster.
After a verify stop, re-entry runs the verify again, and a failure gets one fixer on the last rung before the row stops again.
Rationale: the arbiter, a one-shot fixer that commits and then names a goto target, needs the row to judge its fix by the run's own checks.

### D8 — A run journal carries memory to later batches and the arbiter

Each batch report carries a `journal` field: what the batch struggled with and what later batches must keep in mind.
Go appends it, under a heading naming the batch and its attempt, to `journal.md` in Weaver's run directory; failed attempts' entries are kept.
The append is keyed by that heading and skipped when the journal already holds it, so recording a report again after a crash (D12) adds no second entry.
Every later attempt's prompt and every fixer's prompt names the journal for the agent to read first.
The arbiter's description of a tapestry run names the active implementer's run directory; for Weaver it names the plan directory, the state file, the reports and the journal, so a stop on any row reaches them.
A report with an empty `journal` is accepted.
Rationale: the operator's trial of a journal in place of a long-lived master (2026-11-15), and the board's call that Weaver's description names where the plan, reports and deviations lie.

### D9 — Weaver's run directory and its commits

Weaver owns `_lym/weaver/`: its state file, its reports directory, `journal.md`, `summary.md` and `outcome.yaml`; prompts and locks go under `.lym/weaver/`.
Both are named through `lymdirs`.
The state file follows `internal/state` and refuses no unknown key.
It embeds the shared run-plan state (D11) and adds Weaver's own fields: each attempt's number, rung, strand identity, outcome and failure cause (D6, D12), the decision entries delivered to its strand (D13), each ended batch's end commit (D1), and the verify fixers' numbers and `gatefixer` records (D10).
`outcome.yaml` is the run's outcome marker, written done when the row returns Done.
After each batch ends, done or failed, Go commits `_lym/weaver/` together with the plan directory, through the same records-commit path Webster's run commit uses, so plan rewrites and the journal never leave the tree unclean.
Weaver never pushes; the landing rows do.
The per-batch commits and the journal reach the remote with the run's own push at the row's end, the push every shed transition makes, Done or Stuck alike, and no push is added inside the producer.
A PR rework round archives `_lym/weaver/` into its own prior-generation slot (D13), so the next generation starts fresh.
Rationale: the journal must outlive a crash (2026-11-15), and a commit per batch puts it on the weft branch.

### D10 — The plan-level verify, its fixer and the run record

After the last batch, Weaver runs the plan-level verify through `verifytree.Verify`, in held gate slots, with the gate key `weaver.yaml`'s `verify_gate` names (default `webster`) and one rerun for flakiness, as Webster's gate does.
A failure starts a cold fixer on the ladder (D5) with the findings and the journal, through `gatefixer.SpawnOrAttach` and `gatefixer.Await`,
with a Weaver-owned `gatefixer.Store` over the fixer records in Weaver's state, as Webster's verify fixer does; the fixer commits `fix: <summary>`, and the verify runs again.
The fixer's reply file is `fix-<n>.md` in Weaver's reports directory, `<n>` its fixer number; it is the strand's only output file and its identity under D12.
`gatefixer` saves the fixer's record only after the spawn returns, a window D12 bounds.
The fixer runs under `gatefixer`'s verify-fixer role with `weaver.yaml`'s `fixer_timeout_min` as its timeout, default 60; batten's watch reads that key for a Weaver run's verify-fixer strand.
The fixer carries no shuttle gates, so the row's `gates` entries ride the batch strands alone (D4); Go's re-run of the verify judges the fixer's work.
The row's `gates` entries do not run over a fixer's commit (Open risks).
One fixer runs per failed verify evaluation, and the fixer's own gate calls spawn none.
Every fixer that ends, done or lost, is followed by the verify again before any next fixer spawns, so a lost fixer's same-rung retry (D6) also follows a re-run.
A verify that refuses the tree a fixer left unclean counts as a failed evaluation whose findings are the uncommitted paths, and the next fixer is told them.
A verify runner crash, a run that ends with no evaluation, gets no fixer and stops the row.
A failure after the last rung stops the row; a pass returns Done.
Go writes `summary.md` in the shared final-summary format: the batches and their outcomes, the flaky identities as `## Integration suite triage`, and each fixer's findings and commits as `## Verify gate fixes`, so Describe carries them into the change description through D13's readers.
Rationale: the implementer contract's Done means the plan-level verify passed; the `webster` gate key spares every hub a new `gate.yaml` entry.

### D11 — Webster's batch machinery is shared, not copied

Weaver reuses Webster's report contract and parser, plan fingerprint and plan-conflict refusal, post-batch checks (done checks, handle binding, scope guard, renames rewriting pending cards), suspect-path checks, failure digest, card gate rendering, strand await and verify-failure record.
The state types the shared pieces read move with them into a package both implementers import: a run-plan state holding the plan fingerprint, plan file hashes, overview frame hash, the recorded partition and the batch records.
Both implementers' state files embed it; Webster's `State` keeps its Master fields beside it, and a batch-record field only Webster fills stays optional and empty under Weaver.
Every told directory of `Geometry` that names Webster's run, reports, prompts or scratch directory names the active implementer's under Weaver, so Weaver's state lock and state-mutation lease sit under `.lym/weaver/`.
The batch list Weaver hands the shared checks is its recorded partition mapped onto the current plan, the same mapping Webster's verbs use, so the later-reference check sees the pending cards and only the run's last batch reads as last.
Under Weaver, every shared refusal names Weaver's way forward: a plan-partition refusal names D7's, never Webster's rebaseline or restore-plan verbs.
Webster's behaviour stays as it is.
Weaver's report adds `journal` to the shared contract; Webster's forks may leave it empty.
Fork transcript audits stay Webster's alone.
Rationale: PATTERN-shed-driver's shared-machinery rule; two copies of the post-batch checks would drift.

### D12 — Crash and resume

The row's producer holds a run lock for its whole call.
Every attempt of a batch, a same-rung retry (D6) and a re-entry attempt (D7) included, takes the next attempt number.
Before it spawns an attempt's strand, Weaver fixes the attempt's strand identity: its report path, unique to the batch and attempt number, which the strand gets as its output file.
State records that identity with the batch, attempt and rung, marking the strand in flight, and records the strand's guid once the spawn returns.
Before it spawns a verify fixer, Weaver records the fixer's number, which fixes its reply path (D10); `gatefixer` saves the fixer's record only after the spawn returns.
Shuttle's attach (`AttachGated`) finds a live, never-terminated run by its declared output files, with no guid, so it finds the strand's run even when state never recorded its guid.
On a new call Weaver first stops the live strand of every attempt and fixer whose outcome state records (D6), before it starts anything.
Then, for every batch attempt state records in flight, it attaches through `AttachGated` with the strand's output file and gates.
The attached Wait settles a run whose report is already on disk under its gates, so a strand still re-prompted on a failing gate is awaited until its gates pass or its re-prompts run out, the latter a D6 failure.
Weaver records a report only from the result that Wait returns.
A report found on disk when the attach answers not found carries no gate verdict, so Weaver counts that attempt a machinery loss (D6) and never records it done.
Before it awaits a strand it attaches, a fixer `gatefixer` attaches included, Weaver sends it every decision entry recorded since the attempt or fixer began that state does not record as delivered to it.
Only then does it treat a run that is gone with no report per D6; a respawn-eligible run the attach clears counts as a machinery loss.
A fixer whose record `gatefixer`'s store holds live is attached by `gatefixer.SpawnOrAttach` and awaited through `gatefixer.Await`.
A fixer number state records with no `gatefixer` record means its spawn never recorded a fixer, through a crash between spawn and record, a crash before the spawn or a failed start:
Weaver probes its reply path through shuttle's `ProbeGated`, stops a strand it finds through the handle's `Stop`, counts that fixer a machinery loss (D6), and runs the verify again before any new fixer spawns.
An `AttachGated` or `ProbeGated` error makes the row return Stuck naming the error and its way forward, and spawns nothing.
Shuttle returns one when a candidate cannot be confirmed dead or alive, when several live runs match, when reed state is unreadable, or when a superseded strand cannot be removed.
Rationale: a row is re-stepped after any crash or move, and a gated strand relaunched after a crash sits idle with its report already written; spawning twice would race two agents on one tree.

### D13 — Loom's readers of implementer state read the active implementer's

The loom paths that read Webster's run state today read the active implementer's through one seam, so each sees Weaver's state when Weaver runs.
The seam picks the implementer whose run directory holds begun batches in the current generation, and falls back to `tapestry.implementer` only when neither holds any.
A missing state file counts as no begun batches.
When both hold begun batches, or either state file fails to load, the seam refuses with D1's ways forward.
The D1 guard, the plan gate, the renumber verbs and Describe take that refusal; the decision-notice guard and merge-in's warning, which only advise, fall back to `tapestry.implementer` on it and log its reason.
Its readers:

- the plan gate's done-card freeze (`internal/loomshed/gates.go`) and the Plan-Review rubric's frozen-card note (`internal/loomshed/publishfailure.go`);
- the renumber verbs' bound at the last begun card (`internal/loomcli/renumber.go`), which take the active implementer's state-mutation lease around the bound check and the renumber, the lease Weaver records a batch begin under (D3);
- the plan-conflict record and its route claim, which move to one implementer-neutral path both implementers write, named through `lymdirs`, and which the tapestry recipe's Plan-Review text names instead of `.lym/webster/plan-conflict.md`;
- the decision-notice guard (`internal/loomcli/decision.go`), which treats Weaver's batch and fixer strand roles as implementer strands that get the inline decision while their run runs, and records each delivery in Weaver's state under its state-mutation lease; a halted run's Weaver strand is held back, as Webster's is, and D6's stop removes the strand of every attempt that ended, so none acts beside the arbiter; D12's attach sends a strand still in flight the decisions recorded and not delivered meanwhile;
- Describe's live run record (`RunRecordPath`) and its prior-generation readers (`ArchivedWebsterDirs`), which read the active implementer's run directory and every archived implementer slot of each rework round;
- the PR rework archive seam, which archives the active implementer's run directory, `_lym/weaver/` into its own `prior-generation/weaver` slot that Describe's prior readers list;
- `lym fabric merge-in`'s in-flight warning (`websterengine.RunInFlight`), which reads the active implementer's state and, as for Webster, counts a Weaver run in flight while its state file exists and its `outcome.yaml` (D9) is absent or not done.

Rationale: a reroute to Plan-Burler-Review after Weaver has built cards must freeze them and see the conflict, or the plan is rewritten under landed work.

### Added after Discussion (arbiter, 2026-11-16): One producer runs every batch (answers F11, Q17)

Decision: The record's reading stands: one Weaver row whose producer loops over the batches in Go, each batch's stop returned as the row's Stuck so it reaches shed's stop chain (rule table, tackler, arbiter, parent), meets the operator's 2026-11-15 decision that each batch is a producer. The writer states this in D2 as settled, drops the first Open risks line and marks Q17 answered by this decision.

Rationale: The operator's decision names its purpose: the Shed driver's tackler and arbiter handle a stuck batch as for any recipe, which D2's per-batch Stuck returns give. The board's own 'Weaver as a Shed run' idea lists one batch producer looping over the batch index, as the Burler round loops over rounds, as a way to meet it; the 2026-10-17 operator view puts the batchifier inside Weaver rather than in rows of its own; the implementer seam pins one row with fixed edges, and recipes are fixed files.

### Added after Discussion (arbiter, 2026-11-16): The journal is pushed with the run at each shed transition (answers F10, Q16)

Decision: Weaver keeps the implementer contract: it never pushes, and no push is added inside the producer. Its per-batch commits of _lym/weaver/ and the plan directory (D9), the journal included, and the batches' code commits are pushed by the run's own push at the row's end, the push every shed transition makes, Done or Stuck alike. The writer states this in D9 after 'Weaver never pushes', drops the second Open risks line and marks Q16 answered by this decision. push-before-act-takeover's scope is not changed.

Rationale: The board says the journal is pushed with the run, that is by the run's push alongside the code it describes, and outlives a crash or a move to another machine. A crash keeps the per-batch commits locally, and a move to another machine goes through a stop: the adopt redesign's run FSM leaves every stopped run in a pushed state another machine goes on from, and a Weaver stop is a shed transition that pushes both sides. A push from Weaver would break implementer-spec's never-pushes line, and a push inside a producer would widen push-before-act-takeover, another task's scope.

### Added after Discussion (arbiter, 2026-11-16): The row's gates entries need not run over a verify fixer's commit (answers Q18)

Decision: D10's bound stands: a verify fixer runs through gatefixer with no shuttle gates, and the row returns Done when the plan-level verify passes after the last fixer, with no gated strand added over a fixer's commit and no PATTERN-gate-slot change. The Open risks question and support-log Q18 are answered by this decision; the plan writer builds D10 as written and adds no extra spawn for the row's gates.

Rationale: implementer-spec.md pins the implementer's Done as 'the plan is built and the plan-level verify passed', which D1 keeps and D10's Rationale cites; the row's gates entries are not part of that contract. PATTERN-gate-slot, as the record's Constraints line states it, runs a cold verify fixer only through gatefixer, so the review's option (b) would need a PATTERN change the record chose against in round 3. The one gates entry the shipped tapestry recipe gives the implementer row, drift, is repeated downstream by the Drift-Verify row after Parent-Align as the guarantee (contracts/recipes/tapestry-recipe.yaml), so a drift the fixer's commit causes is still caught before review. An extra gated strand over each fixer commit would add a spawn and its own crash and retry rules to a path the review just closed.

## Constraints

- PATTERN-shed-recipe-registry: the `Implementer` entry's closed table is the one exception to the registry map; Weaver joins that table and adds no registry entry.
- PATTERN-shuttle-provider-seam: rung specs reach a spawn only as `ModelChoice` built by `NewModelChoice`; strands stop only through shuttle's stop verb; a negative finish consults the output files first.
- PATTERN-reed-panes: every batch and fixer strand is a reed session started through shuttle, never headless.
- PATTERN-fabric-boundary: the per-batch commit of `_lym/weaver/` and the plan directory goes through fabric, as Webster's run commit does.
- PATTERN-config-layers: `weaver.yaml` is a new module loaded through `configengine` and registered in `configreg`; its template ships the defaults.
- PATTERN-lym-files: `_lym/weaver/`, `.lym/weaver/` and the shared plan-conflict path are named only by `lymdirs`; locks through `lock`.
- PATTERN-tolerant-records: Weaver's state file never refuses an unknown key.
- PATTERN-gate-slot: the plan-level verify runs in held gate slots through `verifytree.Verify`; its cold fixer runs only through `gatefixer`, one per failed evaluation, none for a runner crash, and Go's re-run judges it (D10).
- PATTERN-stencil-ownership: Weaver's prompts are stencils read at call time through `stencilstore`, rendering the parent and edit directives.
- PATTERN-refusal-way-forward: every Weaver refusal and Stuck reason names its way forward, and a test reaches it.
- A Codex spec may stand on a rung only as far as `codex-same-as-claude` has given Codex sessions what Claude sessions in the same role get; the default ladder is Claude-only.

## Auto-mode assumptions

- The loop over batches lives in the row's producer (D2).
- Batches run sequentially in the task worktree (D4).
- The default ladder is `sonnet[medium]`, `opus[medium]`, `opus[high]` (D5).
- A machinery loss retries once on the same rung (D6).
- A re-entered ladder stop gets one attempt on the last rung; a re-entered `NEEDS_PARENT` stop resumes at the rung that asked (D7).
- The journal is a report field Go appends (D8).
- The verify reuses the `webster` gate key by default (D10).

## Open risks

- Under D10 a verify fixer runs through `gatefixer`, which gates no strand, so the row's `gates` entries do not run over a fixer's commit. The row can then return Done with a row gate failing at HEAD, when that gate checks something the verify gate does not. Question for the parent: accept that bound; spawn the fixer through shuttle with the row's gates under a PATTERN changes entry for PATTERN-gate-slot; or run one gated last-rung strand over a fixer's commit before Done, as D7 does for a stopped batch?

## Acceptance criteria

- With `tapestry.implementer: weaver`, the tapestry recipe builds the Webster row with Weaver, the row accepts its `gates` key, and its Done, reroute and Stuck routes match Webster's (D1).
- The row refuses when the other implementer has begun batches in the current generation, naming `lym webster reset --to start` or `lym weaver reset` by direction; a test runs each named step and the row then proceeds; a corrupt state file of either implementer refuses, naming its repair; for Weaver's it names restoring the last committed version only while the task branch's HEAD is that version's last batch end commit and no Weaver batch or fixer strand is live, and `lym weaver reset` otherwise (D1, D2).
- `lym weaver reset` refuses while Weaver's run lock is held, and, before it changes anything, while a probe by recorded output file finds the run of a batch attempt or fixer, naming each way forward and stopping no strand it finds; the probe removes the strand of an attempt whose outcome is recorded but which was not yet stopped, and finds one whose spawn returned with no guid recorded; over a corrupt state file it refuses while reed lists a live Weaver batch or fixer strand; with `--stop-strand` it stops every strand it finds and then resets; otherwise it moves the task branch back to the run's start commit, restores the plan directory in the weft so the plan's pending cards read as at the run's start, renames the run record's files under stamped names inside `_lym/weaver/` that no reader reads, and commits both through fabric, leaving Webster's directory untouched (D2).
- A three-card plan the batcher splits in two runs two batch strands in order, each on the first rung, and the row returns Done after the verify passes, with no other LLM spawned (D2, D4, D10).
- After a batch that reports a rename, the next batch is partitioned from the rewritten pending cards; done cards are matched by slug; a batch's card set stays fixed across its attempts (D3, D11).
- A batch that removes a symbol a pending card names is refused by the later-reference check under Weaver, and only the run's last batch renders as the last batch (D3, D11).
- A batch prompt names the batch's cards, the decision record and the journal; a retry's prompt adds the failure digest and uncommitted paths; the strand carries the card gates and the row's gates (D4, D8).
- A failed attempt starts the next attempt on the next rung's model; a failure on the last rung returns Stuck whose reason names the batch, the state file, the report and the journal (D2, D5, D6).
- A `FAILED` report, a failing gate and an unclean tree each climb the ladder; every attempt that ends without being recorded done has its outcome recorded and then its strand stopped before the next attempt starts or the row returns, and a machinery-loss retry and a Stuck return on a Weaver fault each find the attempt's strand stopped, also when the state write fails; a machinery loss retries once on the same rung, for a batch attempt, a re-entry attempt and a fixer strand alike; a failed stop returns Stuck naming the strand's guid and `lym reed remove`, and starts no next attempt; a `NEEDS_PARENT` report returns Stuck at once with its need (D6).
- An empty ladder or an unresolvable spec refuses at row build, naming `lym config weaver` (D5).
- Re-entering after a batch stop whose batch passes the post-batch checks records it done with no spawn when the row has no `gates` entries, also after the arbiter edits a pending card; one that fails, or whose row has `gates` entries, gets one last-rung attempt, which carries an edit to the stopped batch's own card text; a plan that lost a recorded card or gained one no batch records refuses, naming its ways forward; re-entering after a `NEEDS_PARENT` stop starts the next attempt on the rung that asked and never records the batch done with no spawn; re-entering after a verify stop reruns the verify (D7).
- Each batch's journal entry, failed attempts' included, is appended to the journal once, also when a crash falls between the append and the state write; `_lym/weaver/` and the plan directory are committed after each batch (D8, D9).
- The arbiter's description of a run with Weaver active names Weaver's run directory and journal (D8).
- A verify failure starts one fixer through `gatefixer` on the ladder, under `fixer_timeout_min`, and reruns the verify after every fixer, a lost one included, before the next spawns; a tree a fixer left unclean hands its uncommitted paths to the next fixer; a verify runner crash starts no fixer; a failure after the last rung returns Stuck; `summary.md` carries the triage and fixes sections (D10).
- A call after a crash stops the live strand of an attempt or fixer whose outcome is recorded; attaches through the recorded output file to a live run of a batch attempt recorded in flight, even when state never recorded its guid; records a report only from the attach's result, and counts a report found on disk with no run behind it as a machinery loss; sends the attached strand the decisions recorded since its attempt began and not yet delivered to it; and never spawns a second agent for one attempt; a crash while a strand is re-prompted on a failing gate attaches to that strand, and the batch is not recorded done until its gates pass; a crash during a fixer attaches to its live `gatefixer` record, and a crash between a fixer's spawn and its record stops the strand its reply path finds and reruns the verify, so neither spawns a second fixer beside a live one; an attach or probe error returns Stuck naming it and spawns nothing; a same-rung retry and a re-entry attempt each get a new attempt number (D12).
- With Weaver active, the plan gate freezes Weaver's done cards, also after `tapestry.implementer` is flipped to `webster`; the renumber verbs stop at Weaver's last begun card and a renumber racing a Weaver batch begin is serialized by the state-mutation lease; a plan conflict reaches Plan-Burler-Review; a decision notice is held back from a halted run's leftover Weaver strand; a corrupt Webster state file under a Weaver run blocks neither a decision notice nor merge-in's warning (D13).
- With Weaver active, Describe reads Weaver's `summary.md` and its archived prior-generation records, a PR rework round archives `_lym/weaver/` into `prior-generation/weaver`, and `lym fabric merge-in` warns of a Weaver run in flight and not of one whose `outcome.yaml` reads done (D9, D10, D13).
- Review: `contracts/specs/implementer-spec.md` lists `weaver`, allows commits during the run and names the stop chain; the tapestry Plan-Review text names the shared plan-conflict path; Weaver's package doc states its loop, ladder, journal and run directory (D1, D2, D5, D8, D9, D13).
