# Pause holds no push lock and skips a pair with no reed session

A pause no longer leaves a process waiting while it holds a push lock, and `lym tender pause` and `lym tender status` no longer fail on a pair with no reed session.

## Why

With a pair paused, a `lym batten run` took `fabric.push.lock`, then waited on the pause while still holding the lock.
Every other pair's batten run then failed with `push lock busy`, until the holder was killed.

After a hard power-off, `lym tender pause` and `lym tender status` failed on a pair whose reed session was gone, with `no reed session`.
That pair has no running turn to interrupt, so the failure only blocked the pause.

## What changes

### Push lock (`internal/fabricengine`)

- Every taker of a push lock, `fabric.push.lock` or the board checkout's, now passes the process's pause gate before it takes its first lock.
  It does so through `pauserecord.Admit` under the new operation name `push-lock`, and stays admitted until it releases the last lock.
  A pause therefore holds a pusher only while it holds no lock, and another pair's push takes the lock meanwhile.
- The gate sits ahead of the outermost lock each entry point takes:
  `coalescePush`, `coalesceFlagged` (once per acquire), `PushPairAnchored`, `TakeOverRun`, `retryBoltPush`,
  `Fabric.lockWeftForReplay`, `Bolt.Pull` and `Bolt.PullThenCommitWritten`.
- A pause that starts while a pusher holds the lock reads its `push-lock` operation as in flight and lets it finish.
- The detached push's deferral now fires at the gate before an acquire, never under the lock.
  It puts back the pending flag of every owed side, since after an earlier step pushed, a flag may name a side with nothing left to push.
  `pushDeferral` lost its `pushed` map and `coalescePushSides` its bookkeeping.

### Pause and status (`internal/tenderengine`)

- `coveredSessions` treats a `reedengine.ErrNoSession` from `Pair.Sessions()` as "no running turn".
  `Pause` keeps the entry, reports that pair under `Skipped` with reed's refusal as the reason, and goes on.
- `Status` gains a `Skipped` list, with the prime included, and still reads every other pair's sessions.
- `Revive` reports such a pair under its `Skipped` the same way.
  The task did not name this verb; it shares `coveredSessions`, so leaving it out would keep the same failure there.
- `docs/overview.md`, the package docs and the `lym tender` help text describe the new reports.

### Dobby

No dobby code changes.
`dobbyengine` already gives a paused run no pass (see its package doc), and the batten driver is the process the lock fix above covers.

## Tests

- `internal/fabricengine`: integration tests pin that a call held by a pause holds no push lock, per entry point (`pushdeferral`, `pull`, `boltpullfirst`, `takeoverrun`, `teardownlock`), and the deferral's flag restore.
- `internal/tenderengine`: `pause_test.go` and `resume_test.go` cover pause, status and revive over a pair with no reed session, naming it as skipped.
- `internal/boardcli/syncdeferral_test.go` is updated for the new deferral.

## Check by hand

Not run:

- The scenario that started this fix, end to end: pause one pair, let dobby restart its batten run, and run `lym batten run` for another pair.
- `lym tender pause` and `lym tender status` on a hub after a real hard power-off.
- The "dobby starts no driver for a paused pair" item rests on the existing rule above; no new test pins it in this change.
