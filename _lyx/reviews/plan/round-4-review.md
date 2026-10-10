---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/13-bolt-push-seed-drop-retry.md
    summary: the seed-commit drop in PushRecorded and in pullFirst holds only the board write lock, not the gitrepo push lock that serialises pull --rebase in _board, so a concurrent pusher's rebase can interleave with the reset and replay
---

### [MEDIUM] The drop's reset and replay are not serialised against a concurrent pusher's `pull --rebase`

**Location:** `_lyx/plan/13-bolt-push-seed-drop-retry.md` (`Bolt.PushRecorded`'s Intent), and `_lyx/plan/07-bolt-pull-first-seed-drop.md` (`pullFirst`'s Intent).

**Issue:** Card 13 has `PushRecorded` take the board write lock (`board.lock`), run `dropSeedCommits` (`ResetKeep` to upstream, `CherryPick` per non-seed commit, and on failure `CherryPickAbort` plus `ResetKeep` back to the original tip), then release the lock and retry `pushWeftAt`.
Card 7's `pullFirst` likewise runs `Fetch`, the drop and `MergeFFOnly` under `board.lock` alone.
But `pull --rebase` in `_board` runs under a different lock.
`gitrepo.PushCoalesced` (`internal/gitrepo/push.go:191`) holds `.gitrepo-push.lock` (`gitrepo.PushLockFileName`) for the whole `pushWithRebaseRetry`, including its `pull --rebase` and abort.
It never takes `board.lock`.
Every Bolt push reaches it through `pushWeftAt` (`internal/fabricengine/recordsgit.go:347`): board sync's step pushes after `commitDirty` has released `board.lock` (`internal/boardengine/sync.go`), and `configcli`, `hubreconcile`, `runReconcile`, `runShortname` and `CloneAndWire` push outside any board lock.
So a detached `lyx board sync` already past its commit step, or any of those pushers in another process, can be mid-`pull --rebase` in `_board` while this process resets HEAD and cherry-picks onto it.
The drop's failure path then runs `CherryPickAbort` and a `ResetKeep` to its saved tip in the middle of the other process's rebase.
That is the destructive state the chokepoint's checks exist to rule out, and none of the four checks (containment, ownership, dirtiness, force) sees an in-progress rebase.
A commit made under `board.lock` today has the same exposure, but a commit only adds; a reset and replay can strand the other pusher's rebase or rewind what it rebased.

**Fix:** Have both drop callers also hold `gitrepo.PushLockFileName` in the board dir for the whole drop, taken after `board.lock` (so the order is `board.lock` then the push lock).
That order cannot deadlock against board sync, which never holds the push lock while it waits for `board.lock`.
`PushRecorded` takes it after `pushWeftAt` returns, since `PushCoalesced` has released it by then.
State the lock in cards 7 and 13's Intent and in the `doc.go` text they plan.
Card 7's integration tests need no new top-level test; a subtest that holds the push lock and asserts the drop waits would pin it.

## Focus settlements

Both focus entries were reached; neither is a defect.

Entry 1, `NewMutations` handlers that call `Bolt.Push`: grepping `internal/fabriccli` and `internal/fabricengine` turns up three handlers that build a record and push Bolt.
They are `CloneAndWire` (`internal/fabriccli/clone.go:48`, push at `:78`), `runShortname` (`internal/fabriccli/shortname.go:48`, push at `:53`) and `runReconcile` (`internal/fabriccli/fabric.go:721`, pushes at `:774` and `:842`).
All three are on card 13's `Edit:` list, and card 13 moves each of those pushes to `PushRecorded` with the handler's record.
No `fabricengine` function calls `Bolt.Push` or `pushWeftAt` besides `Bolt.Push` itself.
The pair verbs in `weft_verbs.go` push the records sibling through `PushPairAnchored` and `CoalescePushBothAt`, not `_board`.
Card 13 names three recordless callers: board sync, `hubreconcile` and `configcli`'s hub-wide edit.
They are the only other `Bolt.Push` sites (`internal/boardengine/sync.go`, `internal/hubreconcile/walk.go`, `internal/configcli/hubwide.go`), and none of those packages builds a mutation record.

Entry 2, `PullThenCommitWritten` callers with no envelope: card 8 passes a local `NewMutations(hub)` and card 10 a local `NewMutations` rooted at the board dir's parent.
Each card states that the drop is logged inside Bolt because the caller has no envelope.
Card 7 makes `pullFirst` log every drop at Info beside its `KindCommitsDropped` entry.
That matches card 13's rule for recordless push callers: `Bolt.Push` logs the dropped SHAs from its local record.
Every path that can reset `_board` therefore either carries the drop in its envelope (cards 11 and 13) or logs it (cards 7, 8, 10 and 13's `Bolt.Push`).
