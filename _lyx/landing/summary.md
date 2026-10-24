# Webster: let a batch wait on its parent without spending recovery

A Webster batch whose work needs a step only the parent (the orch) can take now waits for the parent instead of burning its recovery budget and halting for a human.
Before this, a fork that needed a hub setting such as `gate.yaml`, a refused mutating `lyx fabric` verb or a board write reported `FAILED`.
`recover-batch` read that as a broken card and answered with a cold recovery that could not take the step either, so the batch ended dead and the run halted blocked.
Re-entering the halted run then appended further stuck entries to the history without progress.

Closes #867

## What changes

- **`NEEDS_PARENT` report status.**
  The fork-return report gains a third status with a required `need` line naming the step.
  `ParseReport` refuses the status without a `need` and a `need` on any other status.
  The implementer body tells a fork or recovery strand to use it for steps outside its session's reach, and maps the shared "reports status: FAILED and the orch runs it" clause to it.
- **Waiting batch.**
  `record-batch` and `recover-batch` record such a report as a non-terminal batch with status `waiting`.
  The report is archived, `CurrentBatch` is kept, no post-batch pass runs and no recovery is counted.
  `record-batch` on a waiting batch refuses, naming the need and `begin-batch NN`.
- **Resume.**
  A fork batch resumes through `begin-batch NN`, which already keeps the start SHA, the recovery count and the committed work.
  A recovery batch resumes through `recover-batch NN`: the attach path skips a waiting record and the one spawn that replaces it is not counted, even at the cap.
  A strand that reports `NEEDS_PARENT` again only sends the batch back to waiting.
- **Master.**
  On a `waiting` envelope, Master messages its parent once with the batch and the need, then ends its turn and waits.
  It resumes the batch when the parent says the step is done, and ends the run stuck naming the need when the parent says it will not be taken.
  `recover-batch` exits 0 on a waiting digest.
- **Recovery verdicts name their strand.**
  A dead or stuck `recover-batch` envelope carries `recovery_session`, and its message names that session.
  The two refusals the issue called contradictory came from two different strands with a batch-start reset between them; the session tells them apart.
  The retry rule itself is unchanged.
- **Run entry refuses a dead-end batch.**
  `lyx webster run` refuses, before spawning a Master, a run with a batch that is dead with no retry or whose recoveries are exhausted.
  The error wraps the new sentinel `ErrBatchDeadEnd`, and its way forward is the batch-start reset, then the re-entry step, or the reset to start.
  The Webster shed adapter maps it to `Stuck` like the other operator-only halts.
- **Repeated halts fold.**
  A shed `Stuck` that blocks at its own row because the row has no `OnStuck` is recorded budget-exempt, as a `Halt` Stuck already is.
  An identical repeat folds into the existing history entry (`repeats`) instead of appending another.

## Design decisions

- No new shed halt state or anomaly kind: the board entry chose a waiting state inside Webster that spends no recovery budget and tells the orch.
- A waiting batch is deliberately non-terminal, so every in-flight reader (dispatch scope, reclaim, audit) behaves as before.
- The dead-end refusal fires only on states where Master's own ladder already ended the run stuck at once, so it removes a Master spawn and changes no outcome.
- The budget-exempt flag changes no budget: a blocking row never reaches the budget comparison, and a goto or reroute between two halts still keeps them separate entries.
- The new `need` and `recovery_session` keys are additive, so records written before them read unchanged.
- Webster stays fabric-blind.
- The master template and implementer body change in `contracts/stencils/webster/`; `internal/websterengine/doc.go`, `internal/shedengine/doc.go` and `internal/shedadapters/doc.go` carry the built design.

## Not covered

- The generic "reports status: FAILED and the orch runs it" clause other roles read is unchanged.
- The begin-batch `plan_conflict tree_moved` refusal the issue mentions is a separate observation and is not touched.

## To check by hand

- A Master held on a waiting batch is still bounded by `master_timeout_min`; a parent slower than that ends the run on the timeout, and re-entry re-drives the batch.
  This was not exercised against a live run.
- Whether implementers over-use `NEEDS_PARENT` for failures they could fix has not been observed; the implementer body limits it to steps outside the session's reach.
