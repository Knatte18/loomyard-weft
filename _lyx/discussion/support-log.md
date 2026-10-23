# Support log — orch-compact-repeats

## Interview

A darn run holds no interview: the writer worked from the board entry and a reading of the code.

## Rejected alternatives

- Make creel's withdraw also remove an inject a delivery attempt holds: rejected, since the attempt may already have typed it, and the watcher can reach the same safety by reading the outcome again.
- Withdraw nothing and wait for the `/compact` outcome to settle before every completion: rejected, since a queued `/compact` the session never takes would hold the reload until its time to live ran out.
- Change the reload sequence to skip the color and plugins steps when the session-start hook delivered the pointer: rejected, since the stuck `resuming` state came from the stale `/compact` ahead of the color step, not from the step order.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Is the reported second `/compact` already fixed by `e634efb9`? Self-pick: its draft-refusal requeue is; the withdraw that skips an in-flight inject is not, and is this task's change.
- Q: Should the wait for an in-flight inject show in `lyx orch status`? Self-pick: no; it lasts one delivery attempt, so a Debug line is enough.
