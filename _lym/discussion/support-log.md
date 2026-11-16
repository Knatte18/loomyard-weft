# Support log — weaver

## Interview

The session ran autonomously, so no operator answered; each turn below is an exploration result and the pick it led to.

### Turn 1 — the board entry and the board

The entry carries the operator's decisions of 2026-11-15: Weaver reuses Shed, the arbiter takes Merriam's place, a run journal replaces a long-lived master, and parallel batches come later.
No `Decision for weaver:` note is on the board.
`arbiter-one-shot-fixer` landed as ab5d49e0; the implementer seam landed (`tapestry.implementer`, closed table with `webster` only).
No other board entry plans implementer work; `push-before-act-takeover` owns pushing run records.

### Turn 2 — exploration

Four research subagents reported on the implementer seam and batcher, Webster's batch machinery, shed recipes and stops, and model specs with cold spawns.
Facts the design rests on:

- `internal/batcher` partitions contiguous cards by cost; Webster records the partition once in state.
- Webster's recovery strand (`recover-batch`) is a cold shuttle strand with a failure digest and uncommitted paths, on the `recovery` model.
- Shed's `Yield` ends the driver's loop (`internal/shedengine/run.go`), so it cannot step a producer through batches.
- A row with no `on_stuck` that returns Stuck goes to the tackler, the arbiter, then the parent; the arbiter may now commit one fix and must name a goto target.
- No model ladder exists anywhere; burler's per-round model list is the nearest shape.
- The kick-start pack was removed; only stale config keys remain.

### Turn 3 — auto-picks

Every design question was decided by self-pick; see the question ledger.

### Turn 4 — lens review

Six forks reviewed the record; every one returned a report.

- walkthrough + existing-paths (BLOCKING): the plan-conflict record, done-card freeze, rubric note and renumber bound read only Webster's state — fixed: new D13 routes them through the active implementer and moves the conflict record to a shared path.
- bound-breaker + interactions + walkthrough (BLOCKING/MEDIUM): D7 recorded a stopped batch done on card gates alone — fixed: D7 runs the full post-batch checks against the batch's start commit, and says what an arbiter fix without renames meets.
- interactions (MEDIUM): D5's attempt bound ignored D7's re-entry attempt — fixed: D5's bound counts it.
- interactions (MEDIUM): re-entry after a verify stop was undefined — fixed in D7.
- interactions (MEDIUM): a relaunched gated strand could be attached before its finished report is recorded — fixed: D12 records a finished report first.
- bound-breaker (MEDIUM): a crash between spawn and state write could spawn twice — fixed: D12 records the attempt before spawning and attaches through `FindRun`.
- walkthrough + bound-breaker + interactions (MEDIUM/LOW): the row's `gates` entries had no runner — fixed: D4 and D10 attach them to every batch and fixer strand; D1 says the constructor takes `gates`.
- bound-breaker (MEDIUM): switching implementer mid-run was unguarded — fixed: D1's guard.
- bound-breaker (MEDIUM): machinery losses climbed the ladder — fixed: D6 retries once on the same rung.
- existing-paths (MEDIUM): the per-batch commit left out the plan directory — fixed in D9.
- existing-paths (MEDIUM): PR rework never archived `_lym/weaver/` — fixed in D9.
- existing-paths (MEDIUM): no `summary.md` lost Describe's triage and fix items — fixed: D10 has Go write `summary.md`; Q11 reversed.
- existing-paths (LOW): decision notices treated only Merriam as the implementer — fixed in D13.
- existing-paths (LOW): reportengine's timing phase and failuretrace are Webster-keyed — partly fixed: the verify-failure record is shared under D11; the timing phase rejected, since a report gap does not change a run.
- board-scope (MEDIUM, decision): nothing pushes the journal mid-row — kept as an Open risk, rephrased as a question for the parent.
- board-scope (LOW): the arbiter description was swapped for the Stuck reason — fixed: D8 has the description name the active implementer's run directory; the Open risk is dropped.
- bound-breaker (LOW): implementer-spec's Stuck wording says the run halts for a human — fixed: D1's spec edit names the stop chain.
- interactions (LOW): the spec says the run directory is committed only at Done — fixed: D1's spec edit.
- interactions (LOW): done cards not matched by slug — fixed in D3.
- interactions (LOW): D2's Stuck list left out the reroute, and its reason fit no verify stop — fixed in D2.
- interactions (LOW): "non-interactive" against "interactive" — fixed: D4 and the reed-panes constraint say a reed pane with no operator.
- walkthrough (LOW): the per-batch commit named no committer — fixed: PATTERN-fabric-boundary constraint and D9.
- bound-breaker (LOW): re-partitioning cannot see a done card's delete — rejected: the batcher's boundary only splits batches, a done card is already split from the remaining ones, and a pending card naming a deleted target is caught by the drift check at its begin.
- bound-breaker (LOW): the Codex claim was unchecked — fixed: D4's claim cut; a constraint ties Codex rungs to `codex-same-as-claude`.
- leanness (MEDIUM): D2, D3 and D4 argued rejected alternatives — fixed: cut, they stay in Rejected alternatives.
- leanness (LOW): Goal, Scope and D2 restated decisions — fixed: trimmed.
- leanness (LOW): generic constraints — fixed: sole-parsers and code-conventions lines cut.
- leanness (LOW): the second plan-writer note repeated `relevant.md` — fixed: cut.
- leanness (LOW): auto-mode assumptions repeat decisions — rejected: they are one-line pointers already.

## Rejected alternatives

- A row per batch, generated from the plan: shed recipes are fixed files, and the implementer contract pins one row with fixed edges.
- A nested shed run with Batchify and Batch-Run rows looping, seeded by the implementer row: two shed runs in one worktree, a child arbiter and child run ids, for no behaviour the in-row loop lacks.
- Stepping batches through `Yield`: `Yield` ends the driver's loop and leaves the run waiting.
- A worktree per batch in the sequential first version: 10 to 12 s per add plus a merge per batch, with nothing gained until batches run in parallel.
- A new DAG batchifier now: the operator tied it to the parallel step; the existing cost batcher serves sequential batches.
- A kick-start pack per batch: the prompt names the few files the agent reads, which keeps the start lean without a pack to build and keep fresh.
- An agent-edited journal file: two writers on one file; a report field Go appends keeps one writer.
- Copying Webster's post-batch checks into Weaver: two copies drift; the shared pieces move to a package both import.
- A new `weaver` gate key in `gate.yaml`: every hub would need a new entry; the `webster` key is the plan-level verify already.
- Putting the run paths only in Weaver's Stuck reason: a stop on another row, or one the tackler escalates, reaches the arbiter without them.
- Running the row's `gates` entries from Go after the verify: a gates entry is a shuttle gate that re-prompts a strand, and Go has no agent to re-prompt.
- Recording a re-entered batch done on its card gates alone: it skips done checks, handle binding and the scope guard.
- Passing the done cards to the batcher as context: the boundary rule only splits batches, and done cards are already split from the rest.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the loop over batches live? In the row's producer, in Go; stops reach shed's chain. Why: fixed recipes and `Yield`'s semantics rule out a row per batch, and a nested run adds machinery for nothing. Raised under Open risks for the parent.
- Q2 (auto-pick): Fixed partition or batches formed as the run goes? Formed one at a time from the remaining cards with the active batcher. Why: the board's idea, and renames rewrite pending cards between batches.
- Q3 (auto-pick): Worktree per batch? No; the task worktree, sequentially. Why: parallel batches are out of scope, and a worktree buys nothing without them.
- Q4 (auto-pick): The ladder's shape and default? A list of model specs in `weaver.yaml`, default `sonnet[medium]`, `opus[medium]`, `opus[high]`, one attempt per rung. Why: the board's example ladder; a list fits batches and fixer alike.
- Q5 (auto-pick): Does `NEEDS_PARENT` climb the ladder? No, it stops the row at once. Why: a question is not a model failure.
- Q6 (auto-pick): What happens when the row re-enters after a stop? Card gates at HEAD first; pass records done, fail gets one last-rung attempt. Why: the arbiter's fix must be judged by the run's own gates.
- Q7 (auto-pick): Journal format and writer? A `journal` report field Go appends to `_lym/weaver/journal.md`. Why: one writer.
- Q8 (auto-pick): When is the run directory committed? After each batch. Why: the journal must outlive a crash.
- Q9 (auto-pick): Which verify gate? `verifytree.Verify` with `weaver.yaml`'s `verify_gate`, default `webster`, then the row's gates entries. Why: no new hub config.
- Q10 (auto-pick): How does the arbiter find the plan, reports and journal? The Stuck reason names them. Why: tapestry's arbiter description is shared by every implementer. Raised under Open risks.
- Q11 (auto-pick, revised after the lens review): Does Weaver write a `summary.md`? Yes, written by Go from state and the verify record. Why: Describe reads the triage and verify-fix sections from it.
- Q12 (auto-pick): How do loom's readers of Webster state see Weaver's? Through one seam over the active implementer, and a shared plan-conflict path (D13). Why: a reroute must freeze built cards and see the conflict.
- Q13 (auto-pick): May a run switch implementer mid-run? No; the row refuses while the other implementer has begun batches (D1). Why: switching would rebuild or misread built work.
- Q14 (auto-pick): Does a machinery loss climb the ladder? No; one same-rung retry first (D6). Why: the model did not fail.
- Q15 (auto-pick): Where do the row's `gates` entries run? On every batch and fixer strand as shuttle gates (D4, D10). Why: a gates entry is a strand gate; Weaver has no long-lived strand.
- Q16 (open, for the parent): Is the journal pushed mid-row, and by whom? Under Open risks.
- Q17 (open, for the parent): Is "each batch is a producer" met by one producer running every batch? Under Open risks.
