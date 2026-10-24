# Decision record — webster-orch-need-to-parent

## Goal

A Webster batch whose work needs a step only the parent (the orch) can take waits for the parent instead of spending its recovery budget and halting for a human.
A dead batch's recovery verdicts can be traced to the strand each came from, and a re-entry of a run halted at Webster for the same cause keeps the existing halt instead of growing the run's history.

## Scope

- The fork-return report gains a third status, `NEEDS_PARENT`, with a required `need` line naming the step only the parent can take.
- `record-batch` and `recover-batch` record such a report as a non-terminal `waiting` batch, spending no recovery, and Master tells its parent and waits.
- Once the parent has acted, Master resumes the batch where it stood: `begin-batch NN` for a fork batch, `recover-batch NN` for a recovery batch.
- The recover-batch envelope of a dead or stuck recovery names the session of the strand behind the verdict.
- `lyx webster run` entry refuses a run whose batch is at a dead end (a dead recovery with no retry, or recoveries exhausted) before it spawns a Master, with a fixed message whose way forward is the batch-start reset.
- A shed `Stuck` that blocks at its own row is budget-exempt in its history entry, so a repeat of the same halt folds into the existing entry.
- Out of scope: a new shed halt state or anomaly kind, the generic `reports status: FAILED and the orch runs it` clause other roles read, and the begin-batch `tree_moved` refusal the issue mentions as a separate observation.

## Decisions

### A third report status, NEEDS_PARENT

Decision: the implementer body tells a fork or recovery strand that meets a step only its parent can take (a hub setting such as `gate.yaml`, a refused mutating `lyx fabric` verb, a board write) to stop and write `status: NEEDS_PARENT` with a one-line `need`.
`ParseReport` requires a non-empty `need` with that status and accepts none otherwise.
The body maps the shared refusal clause "reports status: FAILED and the orch runs it" to this status for a webster implementer.
Rationale: today such a need is reported FAILED, which recover-batch treats as a broken card and answers with a cold recovery that cannot take the step either.

### A waiting batch is in flight, not terminal

Decision: a `NEEDS_PARENT` report is distilled to a digest with status `waiting` and its `need`.
`record-batch` and `PersistRecoveryTerminal` record it with `Terminal` false and `Status` `waiting`, archive the report, keep `CurrentBatch`, run no post-batch pass and change no recovery count.
`record-batch` on a waiting batch refuses, naming the need and `begin-batch NN` after the parent acts.
Rationale: the batch has not finished; leaving it non-terminal keeps every in-flight reader (dispatch scope, reclaim, audit) as it is, and the archived report lets the re-begin proceed.

### Resume where it stood, at no recovery cost

Decision: a fork batch resumes through `begin-batch NN`, whose re-begin already keeps the start SHA, the recovery count and the committed work.
A recovery batch resumes through `recover-batch NN`: the attach path skips a waiting record, the exhaustion check passes it, and the spawn that resumes it is not counted.
Bound: only a record whose status is `waiting` skips the count, and only the one spawn that replaces it, so a strand that reports `NEEDS_PARENT` again only sends the batch back to waiting.

### Master tells its parent and waits

Decision: on a `waiting` envelope from `record-batch` or `recover-batch`, Master sends its parent one message naming the batch and the need, then ends its turn with nothing running; shuttle's held turn end keeps it until a message.
On the parent's message that the step is done, Master resumes the batch; on one that it will not be taken, Master ends the run stuck naming the need.
A fresh Master that finds a `waiting` batch at resume re-drives it, since the run was re-entered.
`recover-batch` exits 0 on a waiting digest.
Rationale: the board's design choice is a waiting state that spends no recovery budget and tells the orch, with no new halt type.

### A dead recovery's envelope names its strand

Decision: a dead or stuck recover-batch envelope carries `recovery_session`, the session of the strand the verdict is about, and its message names that session.
Rationale: the two refusals the issue calls contradictory came from two different strands with an operator's `reset --to batch-start` between them, each verdict right for its own strand; without the session the records cannot be told apart.
The retry rule itself is unchanged.

### Run entry refuses a dead-end batch

Decision: `Run` refuses, after the pending-audit check and before any Master spawn, when a recorded batch is terminal dead with `RecoveryRetry` false, or terminal and not done with recoveries at the cap and no unrendered amendment.
The error wraps a new sentinel `ErrBatchDeadEnd`, its text is fixed by the batch's record, and its way forward is the batch-start reset then the re-entry step, or the reset to start.
The Webster shed adapter maps it to `Stuck` like the other operator-only halts.
Bound: it fires only on the states for which Master's own ladder already ends the run stuck at once, so it removes a Master spawn and changes no outcome.

### A blocking Stuck folds

Decision: a `Stuck` that blocks at its own row because the row has no `OnStuck` is recorded budget-exempt, as a `Halt` Stuck already is, so `appendOrFold` folds an identical repeat into the existing entry (`repeats`).
Bound: such a row never reaches the budget comparison, so the flag changes no budget; a goto or reroute between the halts still separates them.

## Constraints

- `PATTERN-refusal-way-forward`: the new refusals name their way forward and a test reaches each.
- `PATTERN-tolerant-records`: the new `need` and `recovery_session` keys are additive and a record written before them reads unchanged.
- `PATTERN-stencil-ownership`: the master template and implementer body change in `contracts/stencils/webster/`, read at call time.
- Webster stays fabric-blind; the CLI keeps its commit points.

## Auto-mode assumptions

- The operator is absent; the darn writer picks names: report status `NEEDS_PARENT`, field `need`, digest status `waiting`, sentinel `ErrBatchDeadEnd`.
- The issue's "re-entry of an already-halted escalation" is read as a re-step of a blocked Webster row whose cause has not changed.

## Open risks

- A Master held on a waiting batch is still bounded by `master_timeout_min`; a parent that takes longer ends the run on the timeout, and the re-entry re-drives the batch.
- An implementer may over-use `NEEDS_PARENT` for a failure it could fix; the body limits it to steps outside the session's reach.

## Acceptance criteria

- A fork's `NEEDS_PARENT` report records the batch waiting with its need, archives the report and leaves the recovery count unchanged; a later `begin-batch` re-begins it with the same start SHA.
- `ParseReport` refuses `NEEDS_PARENT` without a need and a need on another status.
- A recovery strand's `NEEDS_PARENT` report records the batch waiting, and the next `recover-batch` spawns a fresh strand without counting it, even at the cap.
- `record-batch` on a waiting batch refuses naming the need and `begin-batch`.
- A dead recover-batch envelope carries the strand's session.
- `Run` refuses a dead batch with no retry and an exhausted batch before spawning Master, and runs after a batch-start reset.
- The Webster adapter maps the dead-end refusal to `Stuck`.
- Two identical blocking Stucks at a row without `OnStuck` leave one history entry with `repeats` 1.
