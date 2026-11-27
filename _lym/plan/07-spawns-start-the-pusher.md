# Card 7 — spawns-start-the-pusher

**Edit:**
- `internal/fabricengine#SpawnDetachedPush`
- `internal/fabricengine#startDetachedPush`
- `internal/fabricengine#pushSpawnCommand`
- `internal/fabricengine#SpawnStatusPush`
- `internal/fabricengine#statusPushRequest`
- `internal/fabricengine#TestPushSpawnCommand`
- `internal/fabricengine_test#TestSpawnStatusPush`
- `internal/fabricengine/doc.go`
- `internal/boardengine#SpawnSync`
- `internal/boardengine#spawnSync`
- `internal/boardengine#TestSpawnSync_RegistersUnderTheHubOwner`
- `internal/boardengine/doc.go`
- `internal/fabriccli/spawn.go`

**Delete:**
- `internal/fabricengine#parentLockHeldBy`

**Uses:**
- `plan:internal/fabriclayout#PusherCommand`
- `internal/fabricengine#Fabric.commit`
- `internal/fabriccli#runAdd`
- `internal/fabriccli/doc.go`
- `plan:internal/pushledger#Read`

**Intent:** D1 and issue #238: every detached push spawn starts or wakes the pusher of its target instead of pushing itself, whichever verb or package calls it.

`SpawnDetachedPush`, and through it `SpawnStatusPush`, `SpawnPairPush` and `Fabric.commit`'s spawn, now launch `lym loom pusher` with `--push-now` for the worktree the request names:
`pushSpawnCommand` returns `fabriclayout.PusherCommand` with `Worktree` the code worktree, `Slug` the pair's slug for a pair and none for the prime, and `PushNow` set, started in the hub's Bolt directory and registered under the hub owner with the push role, as `runpusher`'s spawn is.
`statusPushRequest` still builds the request, so `SpawnResult` reports what was asked, but it no longer sets `PauseSlug`, `TeardownSlug` or `ParentLocked`, which only the push child read;
`parentLockHeldBy` loses its one caller and goes. `lym fabric push` keeps those flags as its own, read from its command line in `teardownlock.go`.
`boardengine.SpawnSync` launches `lym loom pusher --board --push-now` in the board directory instead of `lym board sync --deferrable`; `lym board sync` stays the operator's verb.
So `lym fabric sync`, `lym fabric add`, `lym board sync`'s callers, a tender push, an anchor change and a teardown's records commit all start or wake the one pusher, and none pushes in its own process.
A failed push is the pusher's: it retries an owed one quietly and writes no failed-push record for it (card 5), so no per-push mail reaches the orch.
Bound: a pusher already running for the target is woken, never doubled, since the second process sets the push-now mark and exits at the lock (card 6).
With `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set nothing is spawned, as today.
The docs that name the old child change with it: `internal/fabricengine/doc.go`'s line on `Commit`'s fire-and-forget push, `internal/boardengine/doc.go`'s line on the sync spawned as `lym board sync --deferrable`,
the file comments of `internal/fabricengine/spawn.go` and `internal/boardengine/spawn.go`, and the file comment of `internal/fabriccli/spawn.go` and its `spawnPush` doc.

Tests.
`TestSpawnStatusPush`, tier1, red-first for issue #238, over a hub fixture whose origin is unreachable:
`lym fabric sync`'s commit through `Fabric.commit` with `spawnDetachedPushFn` recording, then the recorded command line run synchronously through the built `lym` binary in the hub's Bolt directory,
until the child exits or the target's ledger entry shows its first attempt, which the test then ends.
On the merge base the recorded child is `fabric push`, whose owed failure writes a failed-push record of class transport, the record a notice mails from;
after the change the recorded line is the pusher's with `--push-now`, the child writes no failed-push record, and the hub's failed-push directory stays empty.
`TestPushSpawnCommand`, tier0, adding under a second to one tier0 run, pins the pusher command lines for a pair, with its slug, and the prime, with none.
`TestSpawnSync_RegistersUnderTheHubOwner` pins that `spawnSync` starts the pusher's `--board --push-now` line and registers it under the hub owner.

**ImpactSummary:** Changes what every detached push spawn starts, for all their callers, and drops one helper only the spawn read; the spawns' signatures and the `lym fabric push` verb stay.
