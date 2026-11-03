# A landing push rides out a busy push lock, and the lock names its holder

A loom run halted at PR-Review with `fabricengine: push lock busy: <weft>/.weft/fabric.push.lock still held after 30s; neither side was pushed` (issue #37).
`fabric.push.lock` is shared by every pusher of a records worktree, so another session's push can hold it past the 30-second wait, and one wait ended the whole run as a halt for the operator.
The error also gave no clue to which process held the lock or for how long.

## What changes

- **The landing push retries.**
  `PushTask`, which PR-Review and Finalize call to push the task branch, now runs the push again while it fails with `ErrPushLockBusy`, after pauses of 30 seconds, 1 minute and 2 minutes.
  Each attempt still waits `StatusPushLockWait` for the lock, so the landing rides out about five and a half minutes of contention before the row halts with the same way forward as before.
  Each retry is logged at Info with the busy error.
  Any other push failure ends the retry at once, so rejections and transport failures keep their current handling.
- **The busy error names the holder.**
  Every holder of `fabric.push.lock` now writes `fabric.push.holder` beside it, with its pid and the time it took the lock, and removes it before it releases the lock.
  That covers the coalescing holder loop behind `PushAnchored`, `PushPairAnchored` and the detached push child, the weft replay's lock, and origin reconcile's try-lock of a weft.
  The `ErrPushLockBusy` error now ends with `held by pid N for 1m35s`, or says the record names a pid that no longer runs, or that no record names a holder.
  A record that cannot be written is logged at Warn and never blocks the push.

## Design decisions

- **No lock to clear for a dead holder.**
  The issue asked to clear a lock whose holder is dead.
  `fabric.push.lock` is an OS file lock (flock, or `LockFileEx` on Windows), which the kernel frees when its holder exits, so a dead process never keeps it.
  A held lock therefore always has a live holder, and the busy report says so; a record naming a dead pid is stale, and the next holder overwrites it.
- **Retry in the landing push, not a driver re-step.**
  Marking `ErrPushLockBusy` transient would make the driver re-step the row once, but `shedtransient` may not import `fabricengine`, and one re-step after 30 seconds rides out less than the retry does.
  The retry also pauses without polling: a single long `AcquireWriteLockWithin` wait would poll the lock every 10 ms for minutes.
- **`commit-records` keeps its 30-second wait.**
  An agent blocks on that verb, and its busy refusal is already a transient way forward the agent acts on, so only the landing push gets the longer retry.

## Tests

- `TestDescribePushLockHolder` pins the busy clause for no record, a live holder, a dead holder and a corrupt record.
- `TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld` pins that the holder loop's record names this process while it holds the lock, that a busy error then names it, and that the record is gone and the lock free afterwards.
- `TestPushRidingOutBusyLock` pins the retry: a busy lock is retried after each pause in turn, and a landed push or any other failure ends it at once.
