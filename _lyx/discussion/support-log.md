# Support log — discussion-scope-in-lenses

## Interview

### Turn 1 — mode and reading

Autonomous mode: no operator answers; every pick is an auto-pick in the question ledger.
Read the board entry, the board, and the entries that touch the task: `umbrella-daemon` (in flight), `landing-escalate-arbiter`, `lens-overlap-audit`, `creel-agents`, `run-verbs-shared`, `loom-beam-darns`, `fixers-on-sonnet`.
Read the loom and darn recipes, `internal/alignment` and `internal/parentreview` docs, the alignment stencil, the lens template, loom's fan keys, the Discussion-Review rubric, the Bouncer's escalation docs, `arbiterstop`, the darn writer stencil, the parent directive, shuttle's hold rule, `lyx loom peers`, `lyx loom circling` and `lyx loom decision add` help, and commits `fc75aef79` and `2e265a4fc`.

### Turn 2 — facts that changed the board entry's premises

- PR-Gate is gone since `fc75aef79`; the alignment agent runs only at `parent-review`. So "keep it at pr-gate" cannot hold, and the agent goes whole (D1).
- The perch's default reviewer fan `review-discussion` holds neither `goal-scope` nor `board-fit`; both run only in the writer's `twill-discussion` forks (D5).
- A `decision` finding does not gate the Bouncer, so the judge converges past it (D8).
- `merge_*` loom keys were retired in `fc75aef79`; the one `merge_*` key the row reads is `landing.yaml`'s `merge_fix_attempts`, and its gate key is `merge-in` (D11).
- `lyx loom decision add` refuses a darn run before its record is committed, so the darn check comes before the record and the answer arrives by message (D10).
- The arbiter can record decisions, and it takes `circling` and `hard-cap` escalations, so a `decision` cause must be refused to it (D8).

### Turn 3 — design picks

Settled D1–D11 as auto-picks; see the question ledger.

### Turn 4 — lens review

All seven forks returned a report; each is saved beside this log as `support-log.lens-<lens>.md`.
Duplicates are merged and named under every lens that raised them.

- interactions + walkthrough (BLOCKING): an escalation over a CONVERGED verdict re-enters through `settledGeneration` as settled — fixed: D8 keeps an escalated round out of the settled generation, with criteria for re-call, `continue` and `accept` over a CONVERGED round.
- interactions (MEDIUM): a writer's Open risks line stays after the parent answers and re-raises `decision` — fixed: D7 makes an answered question no `decision` finding and its Open risks line a `consistency` fix.
- interactions + walkthrough + bound-breaker (LOW/MEDIUM): `accept` drops a LOW or ledger-closed `decision` finding — fixed: D8's accept carries every `decision` finding of the review.
- bound-breaker (BLOCKING): the darn writer's own Decision does not bind darn's Code perch — fixed: D10 adds the later `lyx loom decision add` request and the unrecorded-correction line.
- goal-scope + walkthrough + bound-breaker (LOW/MEDIUM): the darn wait's timeout and respawn lose the answer — fixed: D10's condition is Step 3's record condition, and the parent answers by a decision note that survives a respawn.
- goal-scope + existing-paths + bound-breaker (MEDIUM): `board-fit`'s reads exceed a perch fork's 6 tool calls, and emphasis lenses break the production fan's procedure contract — fixed: D5/D6 rewrite `goal-scope` and `board-fit` as procedures with reads bounded to the cap and a list naming what was read.
- existing-paths (MEDIUM): renaming `NameMergeIn` misfiles archived rows in reports — fixed: D11 keeps the retired name in `reportengine`.
- bound-breaker (MEDIUM): restarting parked `Merge-In` runs ignores the `RenameProducers` precedent, and the claimed refusal is unshown — fixed: D11 carries runs across with `RenameProducers` (verified in `internal/shedengine/doc.go`, "Row renames", and `loomrecipe.DarnRetiredRows`).
- existing-paths (LOW): the row-named fixer report is lost for a run past the row — fixed: D11's reader falls back to the old name.
- existing-paths (LOW): a BLOCKING `decision` finding hits the judgment refusal first — fixed: D8 runs before the refusal.
- existing-paths (LOW): cause validators not named — fixed: Notes name them.
- bound-breaker (LOW): in-flight blocked-at-cap runs and old-binary drivers — fixed: D2's in-flight list.
- goal-scope (MEDIUM): the outdated PR-Gate premise is settled by the writer against D9 — fixed: an Open risks line puts it to the parent; D1 stays the default.
- goal-scope (LOW): silencing the fix-row notice is unlicensed — fixed: D8 names it as a deliberate change to `2e265a4fc`.
- goal-scope (LOW): D4's legacy parser is new carrying cost — rejected: already a recorded auto-pick with its rationale.
- board-fit (MEDIUM): `regression-baseline-verify` relies on the alignment agent — fixed: named in Scope Out among the entries the orch rewrites.
- board-fit (MEDIUM): `loom-beam-darns` pieces on darn would run D10's check — fixed: Open risks line.
- board-fit (LOW): `umbrella-daemon`'s arbiter move must keep D8's refusal — fixed: folded into its Open risks line.
- board-fit (LOW): `creel-agents` and `run-verbs-shared` lose scope — fixed: named in Scope Out.
- board-fit (LOW): `lens-overlap-audit` and `review-setup-audit` baselines — rejected: notes for later audits, no design effect; the fan-growth line moved out of Open risks.
- board-fit (LOW): `landing-escalate-arbiter` text goes stale — fixed: named in Scope Out.
- leanness (MEDIUM): caller-list and stencil-list Notes — fixed: cut; Notes keep only the cause validators and `settledGeneration`.
- leanness (LOW): Constraints restate D2 and D8, and the friction-capture claim is unchecked — fixed: the restart restatement and the friction bullet cut, the registry bullet shortened.
- leanness (LOW): Auto-mode assumptions triplicate the picks — fixed: one line each.
- leanness (LOW): D1's sentence repeated — fixed by the shortened assumptions.
- leanness (LOW): D8's brief detail — rejected: the fork itself kept it under the attack-surface rule.
- leanness (LOW): fan-growth cost line is no risk — fixed: removed from Open risks.

## Rejected alternatives

- Keep `internal/alignment` for a future point: no point remains, and YAGNI.
- Drop archived parent-review and alignment records from reports: silently changes the numbers of audit windows spanning the landing.
- Move `goal-scope` and `board-fit` out of `twill-discussion` into the perch: the writer's cheap self-check catches ordinary misalignment before a perch round; trimming that fan is `lens-overlap-audit`'s question.
- Add `goal-scope` and `board-fit` to `review-discussion` unchanged: a production fan's lenses are procedures, and `board-fit`'s open-ended reads exceed a fork's 6 tool calls.
- Two new lenses split from `board-fit` (board and runs): the board entry names `board-fit` as the lens that reads the runs, and more lenses run against `lens-overlap-audit`'s direction.
- Raise the perch fork's tool-call cap for one lens, or put the board and peers in the reading manifest: both change Burler's shared fork contract for every segment.
- Escalate only on a non-CONVERGED verdict: a CONVERGED judge is exactly the case where the run passes without the parent's answer.
- Put the board check in the rubric so solo rounds run it: every solo round would read the board and peers again; a lens runs it only where the fan says.
- Escalate `decision` findings in every segment: changes the Plan and Code perches, which the board entry does not license.
- Escalate before the judge runs: the escalation brief and ledger come from the judged round, and the judge's spawn is cheap beside a stopped run.
- Escalate only MEDIUM or worse: a LOW `decision` finding would then go neither to the fixer nor to carry-over; D7's tight class makes any-severity safe.
- Keep both the fix-row notice and the Bouncer escalation: two parent messages per round for one question.
- A new finding class beside `decision`: the class exists and already leaves the fixer since `2e265a4fc`.
- Let the arbiter settle `decision` escalations: the board entry routes them to the operator, and the arbiter can record decisions.
- Darn writer checks after writing its record, so the parent answers with `lyx loom decision add` only: the record would already be built on the outdated entry, which is what Part 2 avoids.
- Rename the merge record and fabric's merge-in too: they name git merges, Finalize's included, not the row.
- Run a refused `decision` accept below the hard cap as a `continue`: it forgets the parent's answer, so the next round re-escalates the same question, and nothing hands the refusal's findings to the next fixer; re-judging before the recorded decision acts keeps the answer and the dropped-key guard.
- Halt a no-parent darn stop through a new gate refusal: the existing `discussion` gate already halts once its attempts run out, with no new code.
- Restart runs parked on `Merge-In`: the recipe's restart rule is for retired rows, and `RenameProducers` already carries a one-to-one rename.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Does the alignment agent stay at PR-Gate? — No, it goes whole; PR-Gate was removed in `fc75aef79` (D1).
- Q2 (auto-pick): What happens to a run waiting on a parent-review round at upgrade? — It passes on its next gate evaluation; no restart (D2).
- Q3 (auto-pick): Retired loom keys? — Dropped by reconcile, migrated with a Warn in profiles and `run_settings` (D3).
- Q4 (auto-pick): Archived records in reports? — Kept readable inside `reportengine` (D4).
- Q5 (auto-pick): Which fan carries the scope check? — `review-discussion` gains `goal-scope` and `board-fit`, rewritten as procedures; `twill-discussion` keeps its members (D5).
- Q6 (auto-pick): What do the lenses send to the parent? — Plainly outdated entry, design conflicts with another run and doubt about the task's purpose, with the alignment agent's bounds and reads capped to a fork's budget; ordinary misalignment stays a fixer finding (D6).
- Q7 (auto-pick): How is a writer's choice kept out of `decision`? — The rubric defines the class for this segment (D7).
- Q8 (auto-pick): Where does the escalation live? — A Discussion-Bouncer row key; on the judged round, before the judgment refusal, soft-cap, circling and auto-accept, with an accept still subject to the refusal; the round is no settled generation until its accept settles; cause `decision`, kept on the hard-cap path; arbiter refuses it; accept carries every `decision` finding (D8).
- Q9 (auto-pick): One parent message or two? — One: the Bouncer's escalation; the fix row stays silent in that segment (D8).
- Q10 (auto-pick): What does the Discussion writer do with a parent-only question? — Records it under Open risks in autonomous mode; asks the operator in interactive mode (D9).
- Q11 (auto-pick): Darn check timing and answer path? — While the record is absent or invalid; ask by `SendMessage` for a decision note and a reply, wait held and re-read the board on any parent message, with no parent stop with the change description as stop report, write the answer into the record, then ask for `lyx loom decision add` once the record is committed (D10).
- Q12 (auto-pick): Which names does the rename cover? — Row, engine, producer, fixer mode and report, `merge_fix_attempts`, gate key `merge-in`, docs; not fabric's merge-in nor the merge record; runs carried across with `RenameProducers` (D11).
