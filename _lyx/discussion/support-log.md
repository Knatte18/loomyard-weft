# Support log — plan-card-rules

## Interview

Autonomous run: no operator answered, so every turn below is the writer's own reading of the code and the board, and every pick is an auto-pick in the question ledger.

### Turn 1 — the task and the board

The board entry merges four notes: the plan-gate headroom check (issue #735), the `Extends` label with the `tools/plandag` measurement first, an owner for card amendments (issue #806), and Enclose's first use.
Related entries read: `parallel-batch-parts` and `weaver` (they consume the card DAG; `Extends` must not build their parallel machinery), `shed-arbiter-all` (the arbiter stays loom's seam; D8 uses only its stencil), `plan-writer-context` (no overlap).

### Turn 2 — headroom

`PATTERN.md` is 15,664 bytes on this branch, past the 15,564-byte headroom line (`pattern.HeadroomBytes`, 95% of 16 KiB), so `pattern.Headroom` refuses any growth.
Recorded `## PATTERN changes` bullets carry design prose (for example the `run-process-ownership` record's `PATTERN-run-process-owner` bullet), longer than the overview lines that landed, so the declared text cannot be measured.
Cards can prescribe verbatim entry lines in fenced blocks, which `pattern-entry-line-cap` already scans.
Recommendation taken: project the overview from the verbatim lines and require one for every declared `adds` and `changes` (D1, D2).

### Turn 3 — `Extends`

No production code builds a parallel DAG; the only production dependency rule is Webster's batch-order assertion in `internal/websterengine/sequence.go`, plus `tools/plandag`.
The batcher groups in card order and does not read dependencies.
Recommendation taken: a type label whose refs stay targets for every guard, read differently only by the dependency rule (D3, D4).
The orch's estimate in the board entry already applied the additive guess with plandag's own rule, so the flag lands first and the label does not wait on a rerun (D5).

### Turn 4 — the amendment owner

The Webster row already rebaselines a changed plan on entry, accepting pending cards, cards of failed or in-flight batches and cards after the last begun batch.
Plan-Burler-Fix is the role that edits an approved plan, under the plan gate and Plan-Bouncer's commit seam, and its answer key already handles Webster's `plan-conflict.md`.
`lyx loom goto` refuses a settled segment's review row; Plan-Bouncer over an unchanged plan re-settles with no spawn, so a goto there does nothing for a decision that has not reached the plan.
Recommendation taken: a flag on `decision add` writes an amendment record the settled-segment guard admits, mirroring its Publish failure admission (D7), with the arbiter and the orch routing through it (D8).

### Turn 5 — Enclose

`go.mod` pins quarry v0.4.0, which has `Enclose` and `EncloseAt`; the board entry's v0.2.1 note is stale.
A failed Merge-In or Publish verify writes the Publish failure record, and `PublishFailureNote` renders it into the Webster-Review rubric, which the reviewer and the fixer read.
Locations come from the failure identities' tails: Go build errors carry relative paths, test failures carry bare file names, and panics carry absolute paths, or module-path-prefixed ones under the hub gate tiers' `-trimpath`.
Recommendation taken: trace at record write time, store the cards on the record and render them in the note (D9).

### Turn 6 — lens review

Seven forks reviewed the record; every one returned a report.

- interactions, existing-paths, walkthrough, bound-breaker: D7 claimed Plan-Bouncer's amendment generation, but the review round makes the settle record not live, so Plan-Bouncer judges round by round on the spent budget — fixed: D7 states the real path.
- walkthrough (BLOCKING): D2 refused every PR-Rework generation of a task that declared an entry — fixed: an entry whose bullet already differs from the merge base is satisfied.
- bound-breaker: the amendment record was removed by a no-spawn approved settle — fixed: removal only after a judged round later than the write.
- interactions: a stale amendment record outlives a PR-Rework generation — fixed: PR-Rework's archive removes it.
- interactions: a failed write after the commit left the decision unroutable — fixed: the record is written before the commit and removed on failure.
- leanness (outside lens): a second `--amends-plan` overwrote the first — fixed: titles append.
- existing-paths: the amendment record's location was unstated — fixed: the run's task worktree, located as the decision record is; adoption loss added to Open risks.
- bound-breaker, walkthrough: fenced old or example lines made the projection ambiguous — fixed: last fenced line per card wins, and the stencil fences only the new line.
- bound-breaker: the self-check and the row gate could use different bases — fixed: the CLI gets the same reader.
- bound-breaker: continuation lines — fixed: the whole bullet is replaced.
- existing-paths: `planglyph` delete-order and the batcher boundary read `Edit` only — fixed: D3 names every reader.
- board-fit: the pair rule lived only in Webster and plandag — fixed: one function in `internal/planparser`.
- board-fit: the arbiter stencil is recipe-blind — fixed: the amendment target comes through an arbiter prompt input filled from a recipe key (D8).
- board-fit: C# output unstated — fixed: Scope Out.
- board-fit: `speed-loom-profile` could remove the fix loop — fixed: Open risks.
- board-fit: `run-verbs-shared` moves the verbs later — rejected: no conflict today, and the move rewrites every `lyx loom` mention anyway.
- board-fit: `fixers-on-sonnet` wants concrete findings — fixed: D8 asks for concretely worded decisions.
- goal-scope: the estimate stood in for the measurement — rejected as a plan change: the nine plans live in the records repo outside the worktree; fixed instead by citing the operator scope line as the licence in D5.
- goal-scope: the plan-conflict recurrence halt still names a hand edit — fixed: it names the amendment route; `rebaseline`'s wording moved to Scope Out.
- goal-scope: the orch rule now conflicts with the stencil — fixed: Open risks.
- goal-scope: the trace may often name no card — fixed: merged into one Open risk.
- existing-paths: card intent rendered from the record broke the note's no-free-text rule — fixed: slug and intent are read from the live plan at render time, and the new fields are shape-checked.
- walkthrough: repo-root-relative build-error paths were dropped — fixed: an existing path is taken as written, and `./` paths are placed through the package.
- walkthrough: module path and external test units unstated — fixed in D9 step 1 and step 3.
- interactions: renumbered card ids — fixed: match by live card id; begun cards keep their ids since inserts go after the last begun batch.
- interactions: retired-generation members trace to no card — fixed: Open risks.
- interactions: the flag on a labelled plan was undefined — fixed: labels win.
- bound-breaker: the scope guard is no callable matcher — fixed: D9 states the overlap rule itself.
- leanness: Notes duplicated `relevant.md` — fixed: kept the direction-carrying hints only.
- leanness: the estimate was stated three times, the boolean-flag assumption duplicated D7, the byte tally was perishable, generic constraints and D9 narrative added nothing, two risks overlapped — fixed.

## Rejected alternatives

- Headroom from a byte delta the plan declares: unverifiable, the plan writer would estimate it by hand as the burler round did.
- Headroom checked at Discussion: the record's bullets are prose, not overview lines, so there is nothing to measure there.
- Measuring the declared bullet text as if it were the line: it overstates growth, since recorded bullets run longer than the lines that land.
- `Extends` as a modifier on an `Edit` ref instead of a type label: the card format has no per-ref modifiers, and a label keeps the parser's group model.
- Dropping `Extends` against `Uses:` edges too: a card reading a flag another card adds must still follow it.
- Gating the label build on a plandag rerun inside the run: a loom plan is fixed before Webster, so a conditional build cannot be planned; the estimate already answers it.
- The orch editing cards by role, as the hub orch rule allows: no plan gate or review sees the edit, the plan's commit falls to no row, and the orch role forbids editing a task pair.
- The arbiter editing cards: it would widen a cold judge's write surface to plan files with no review.
- A goto onto Plan-Bouncer: an unchanged plan re-settles with no spawn.
- A card list on `--amends-plan` checked against Webster's state: the decision text names the cards and the reviewer already knows the frozen cards; the extra check adds coupling for a case the plan gate's `done-card-edited` already refuses.
- A kick-start pack of failing members: removed by operator decision (`kickstart-pack-remove`).
- Tracing at note-render time instead of record write: the record's head is fixed when written, and every later render would re-run quarry.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Where is the headroom checked? Plan gate only, by projection from verbatim lines (D1).
- Q2 (resolved, auto-pick): How are declared prose changes handled? Declared `adds` and `changes` need a verbatim line (D2).
- Q3 (resolved, auto-pick): Which base does the plan check use? The merge base with the parent, as the verify gate's headroom lint; `HEAD` with no parent (D1).
- Q4 (resolved, auto-pick): Does `Extends` wait on a plandag rerun? No; the orch's estimate is the measurement, and the flag lands first (D5).
- Q5 (resolved, auto-pick): Is `Extends` a label or a modifier? A type label whose refs stay targets (D3).
- Q6 (resolved, auto-pick): Who owns a card amendment? The Plan-Review fix loop, entered through `--amends-plan` and the guard admission (D7, D8).
- Q7 (resolved, auto-pick): Does `--amends-plan` take card numbers? No (D7).
- Q8 (resolved, auto-pick): Where does the trace run and where does it show? At record write in Merge-In and Publish, shown in the Publish failure note (D9).
- Q9 (resolved, auto-pick): Location cap per trace: 20.
- Q10 (resolved, auto-pick): Does this task bump quarry? No, v0.4.0 already ships Enclose.
- Q11 (resolved, auto-pick): Does this task touch the hub orch rule allowing hand edits? No; it lives outside the repo, and the conflict is an Open risk.
- Q12 (resolved, auto-pick): Should the amendment start a new Plan-Review episode so it gets an allowance? No; the existing goto routes keep the spent budget, and the parent's resume grants more (D7).
- Q13 (resolved, auto-pick): Where does the arbiter learn the amendment target? From an `amendment_target` arbiter prompt input that loom's wiring fills from a recipe key once the plan is approved; the stop envelope is unchanged (D8).
- Q14 (resolved, auto-pick): Where does the `Extends` pair rule live? In `internal/planparser`, for Webster, plandag and a later batchifier (D4).
- Q15 (resolved, auto-pick, round 2): Is the amendment record never-tracked? No; the run's routing reads it, so it is tracked at `_lyx/loom/amendment.yaml` and committed with its decision entry (D7).
