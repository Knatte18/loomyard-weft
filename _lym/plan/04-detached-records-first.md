# Card 4 — detached-records-first

**Edit:**
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#pushSideDetached` -> `func pushSideDetached(rec *Mutations, hubPath, side, path string, opts SyncOptions, retryRecords func() (bool, error)) (landed bool, err error)`
- `internal/fabricengine#coalesceFlagged`
- `internal/fabricengine#CoalescePushRequest`
- `internal/fabricengine#CoalescePushBothAt`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine/doc.go`
- `internal/fabricengine#TestCoalesceFlagged_PendingFlags`
- `internal/fabricengine#TestCoalescePushRequest_CoveredCallDefers`
- `internal/fabricengine_test#TestRemove_TeardownLockAgainstThePairsPush`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`

**Delete:**
- `internal/fabricengine#respawnForeignFlags`
- `internal/fabricengine#foreignFlagRequest`
- `internal/fabricengine#respawnPairPush`

**Intent:**
The detached pushes that stay keep `PushPairAnchored`'s order (D1): the anchor change's `SpawnStatusPush`, fabric's commit spawn, `lym fabric sync` and `add`, and the deferred pair push a lift pays all run `coalescePushSides` in their child.
Its step now pushes the records side first and the code side only once the records side landed.
`pushSideDetached` reports whether its side landed: a recorded failure still returns a nil error, now with `landed` false, and so does a deferral or an ended branch, since `landed` also needs the worktree to hold nothing unpushed afterwards (card 3's check); the step sends no code side after an unlanded records side.
The `pushed` map marks a side only once it is pushed, so a pause deferral puts back the code side's flag when the records side's push was the one the pause cut.

The pending-flag respawn retires (D1):
`coalesceFlagged` no longer respawns a flag of a side the holder does not owe.
Such a flag stays on disk for its worktree's next push, which takes it as an owed side;
in a pair that is the owed retry of the push that set it, which sends both sides through `PushPairAnchored`, records first.
`respawnForeignFlags` and `foreignFlagRequest` go.
`Topology.Remove`'s failure path no longer respawns the pair's push, and `respawnPairPush` goes: the pair's next synchronous push carries the commits.
The fabric doc's pending-flag and teardown-lock paragraphs drop both respawns and state where a left flag goes.
`CoalescePushBothAt`'s doc drops its sentence that it respawns a side another pusher left flagged.

Tests:
- `TestCoalesceFlagged_PendingFlags` (tier0): the rows that expected a validated foreign flag respawned now expect no spawn and the flag still on disk; the rejected-flag rows keep their flag on disk too.
- `TestCoalescePushRequest_CoveredCallDefers` (tier1) drives the real step: the covered call now defers at its first gated push, the records side's, and both flags come back with origin holding neither commit, which fails if the step still pushed code first.
- `TestRemove_TeardownLockAgainstThePairsPush` (tier1): its failed-remove row asserts the push recorder saw no respawn.
- `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` (tier1) gains a row for D1's flagged code side:
  a pair push whose lock wait gave up leaves `pending.code`; a real records-only push of the same records worktree (`CoalescePushRequest` with `WeftPath` alone) then takes and releases the lock;
  origin's code branch stays unchanged, which fails if the holder still respawned the foreign code flag, until a pair push of that worktree lands its records side first.

**ImpactSummary:** Changes the side order of the detached coalesced push and removes two respawn paths and three private functions; no exported signature changes, and the four tests that pinned the respawns are rewritten.
