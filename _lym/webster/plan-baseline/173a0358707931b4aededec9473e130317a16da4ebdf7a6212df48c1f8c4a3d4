# Card 7 — spawns-start-the-pusher

**Edit:**
- `internal/fabricengine#SpawnDetachedPush`
- `internal/fabricengine#startDetachedPush`
- `internal/fabricengine#pushSpawnCommand`
- `internal/fabricengine#SpawnStatusPush`
- `internal/fabricengine#statusPushRequest`
- `internal/fabricengine#TestPushSpawnCommand`
- `internal/fabricengine#TestStartDetachedPush_RegistersTheChild`
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

**Intent:** D1 and issue #238: every detached push spawn starts the pusher of its target instead of pushing itself, or leaves the commit to the one running, whichever verb or package calls it.

`SpawnDetachedPush`, and through it `SpawnStatusPush`, `SpawnPairPush` and `Fabric.commit`'s spawn, now launch `lym loom pusher` for the worktree the request names:
`pushSpawnCommand` returns `fabriclayout.PusherCommand` with `Worktree` the code worktree, `Slug` the pair's slug for a pair and none for the prime, and `PushNow` unset, started in the hub's Bolt directory and registered under the hub owner with the push role, as `runpusher`'s spawn is.
No spawn sets the push-now mark, so the interval in force holds between spawned commits (D2); the mark is the landing's, the settle's and the teardown's (cards 11 and 19).
`statusPushRequest` still builds the request, so `SpawnResult` reports what was asked, but it no longer sets `PauseSlug`, `TeardownSlug` or `ParentLocked`, which only the push child read;
`parentLockHeldBy` loses its one caller and goes. `lym fabric push` keeps those flags as its own, read from its command line in `teardownlock.go`.
`boardengine.SpawnSync` launches `lym loom pusher --board --worktree <board directory>`, the board directory it is given, in that directory instead of `lym board sync --deferrable`; `lym board sync` stays the operator's verb.
So `lym fabric sync`, `lym fabric add`, `lym board sync`'s callers, a tender push, an anchor change and a teardown's records commit all start the one pusher or leave their commit to it, and none pushes in its own process.
A failed push is the pusher's: it retries an owed one quietly and writes no failed-push record for it (card 5), so no per-push mail reaches the orch.
Bound: a pusher already running for the target is never doubled, since the second process exits at the lock (card 6) and the running one takes the commit at its next pass.
With `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set nothing is spawned, as today.
The docs that name the old child change with it: `internal/fabricengine/doc.go`'s line on `Commit`'s fire-and-forget push, `internal/boardengine/doc.go`'s line on the sync spawned as `lym board sync --deferrable`,
the file comments of `internal/fabricengine/spawn.go` and `internal/boardengine/spawn.go`, and the file comment of `internal/fabriccli/spawn.go` and its `spawnPush` doc.

Tests.
`TestSpawnStatusPush`, tier1, red-first for issue #238, over a hub fixture whose origin is unreachable:
`lym fabric sync`'s commit through `Fabric.commit` with `spawnDetachedPushFn` recording, then the recorded command line run synchronously through the built `lym` binary in the hub's Bolt directory,
until the child exits or the target's ledger entry shows its first attempt, which the test then ends.
On the merge base the recorded child is `fabric push`, whose owed failure writes a failed-push record of class transport, the record a notice mails from;
after the change the recorded line is the pusher's without `--push-now`, the child writes no failed-push record, and the hub's failed-push directory stays empty.
A second row, adding about two seconds to one tier1 run, runs `lym fabric add` over the hub fixture with `spawnDetachedPushFn` recording, and asserts each recorded line is the pusher's without `--push-now`, for the target the add's commit went to.
`TestStartDetachedPush_RegistersTheChild`'s pair row expects owner `runproc.HubOwner` and the push role in place of the pair's slug, and asserts the spawned line is the pusher's command line.
`TestPushSpawnCommand`, tier0, adding under a second to one tier0 run, pins the pusher command lines for a pair, with its slug, and the prime, with none;
its pair rows' `wantOwner` becomes `runproc.HubOwner` with the push role, in place of the pair's slug.
`TestSpawnSync_RegistersUnderTheHubOwner` drives the exported `SpawnSync`, the call every `lym board sync` caller makes with the Bolt directory and nothing else, such as heddle's `startSync` and the tender's board sync,
and pins that it starts the pusher's `--board --worktree <board directory>` line, with no `--push-now`, and registers it under the hub owner.

**ImpactSummary:** Changes what every detached push spawn starts, for all their callers, and drops one helper only the spawn read; the spawns' signatures and the `lym fabric push` verb stay.
