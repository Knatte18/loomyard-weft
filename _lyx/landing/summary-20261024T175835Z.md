# Let a Webster batch wait on its parent instead of spending its recovery

A Webster fork that needed a step only the orch can take, such as a change to the hub's tier commands in `gate.yaml`, had only one way to say so: report `FAILED`.
recover-batch then spawned a cold recovery strand that could not take the step either, it ended dead with `asking`, the batch's recovery budget was spent, and the run halted for a human.
Each later re-entry spawned a fresh Master that ended stuck again and added another `Webster / stuck` entry to the run's history (issue #867).

## What changes

**A third report status.**
The fork-return report accepts `status: NEEDS_PARENT` with a required one-line `need` naming the step.
`ParseReport` refuses `NEEDS_PARENT` without a need, and a need on any other status.
The implementer body tells a fork or recovery strand to use it for a hub setting, a `lyx fabric` verb lyx refuses from the strand, or a board write.
It also maps the shared refusal clause "reports status: FAILED and the orch runs it" to this status.

**A waiting batch.**
`record-batch`, and `recover-batch` through `PersistRecoveryTerminal`, record such a report as a batch in flight with status `waiting` and the need in its digest (`internal/websterengine/waiting.go`).
The report is archived, and neither the post-batch pass nor the suspect check runs.
The recovery count and start commit stay unchanged.
`record-batch` on a waiting batch refuses, naming the resume verb.
`recover-batch` exits 0 on a waiting digest.
It never attaches to a waiting record and never refuses one as exhausted.
The spawn that resumes a waiting batch is not counted.
The progress trail lists a waiting batch with its need and its resume verb.

**Master tells its parent and waits.**
On a `waiting` envelope, Merriam sends its parent one message quoting the need and ends its turn with nothing running.
The turn end is held until a message arrives.
When the parent says the step is taken, Merriam resumes the batch where it stood: `begin-batch NN` and a fork for a fork batch, or `recover-batch NN` for a recovery batch.
When the parent says the step will not be taken, Merriam ends the run stuck and quotes the need.
A fresh Master that finds a waiting batch re-drives it.

**Recovery verdicts name their strand.**
A dead or stuck `recover-batch` envelope carries `recovery_session`, and its message names the session.
The issue's two "contradictory" refusals for batch 22 came from two different strands, with an operator's `reset --to batch-start` between them, and each verdict was right for its own strand.
The retry rule itself is unchanged.

**Re-entering a dead end halts without a Master.**
`Run` now refuses on entry, before it spawns Master, while a recorded batch is dead with no retry or has spent its recoveries and holds no unrendered amendment.
The error wraps the new `ErrBatchDeadEnd`, and its text depends only on the record and the tree.
Its way forward is `lyx webster reset --to batch-start --batch NN` followed by the re-entry step, or the reset to start.
The Webster shed adapter maps it to `Stuck`, like the other halts only an operator can clear.

**A repeated halt folds.**
In `shedengine`, a `Stuck` that blocks at a row with no `OnStuck` is now recorded budget-exempt, as a `Halt` Stuck already was.
No budget applies at such a row.
A re-step that blocks the same way again therefore folds into the existing history entry (`repeats`) instead of appending a new one.

## Design decisions

- A waiting batch stays non-terminal (`Terminal` false, `Status` `waiting`).
  Every terminal non-done record reads as "needs recovery" to begin-batch, record-batch and the retry rule, while every in-flight reader already treats the batch correctly.
- A recovery batch resumes through `recover-batch`, not a fork re-begin, so the failed batch's suspect paths and failure digest reach the resuming strand.
- The resume spawn goes uncounted, rather than refunding the count of the spawn that reported the wait, so no record has to say whether that spawn counted.
- The shed change marks the entry budget-exempt instead of skipping the producer call on a blocked row.
  A re-step of a blocked row is still how a human resumes after fixing the cause.
- The begin-batch `tree_moved` refusal that the issue also lists is a separate observation and is not changed here.

## Tests

- `ParseReport` accepts the new status in a round trip, and refuses it without a need and refuses a misplaced need.
- `RecordBatch` records a waiting batch and archives the report, and then refuses on the waiting batch.
- A begin-batch re-begin of a waiting batch keeps its start and its count.
- The recover-batch CLI scenario covers the session on a stuck envelope, a waiting recovery at two recoveries, and its uncounted resume.
- `Run` refuses both dead-end shapes and reaches Master once the reset's effect is applied.
- The adapter maps `ErrBatchDeadEnd` to `Stuck`.
- Two identical blocking Stucks leave one history entry with `repeats` 1.
- The progress trail renders waiting batches.
