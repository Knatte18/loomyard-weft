# Card 10 — `lyx batten status` shows which child waits on which reviewer

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed/innerrun_test.go`
- `internal/battencli#battenCLI.wire`

**Uses:**
- `plan:internal/parentreview#Store`
- `plan:internal/parentreview#Store.WaitNote`
- `plan:internal/loomengine#LoomParentReviewDir`
- `plan:internal/loomengine#LoomParentReviewLockDir`
- `internal/battencli#taskWorktreeLocation`

**Intent:**
Carry the child's reviewer wait into batten's InnerRun wait message (decision D9), so the hub sees it from the prime.

`InnerRunDeps` gains `ReviewWait func() (string, error)`, resolved on Call like `ReadDecision`; nil means no note.
In the running arm of `innerRunProducer.Call`, a non-empty note replaces the plain still-running reason with `inner shed run waiting: <note>; sleeping <interval> before the next bounce`.
A `ReviewWait` error logs one Warn and falls back to the plain reason: the note is advisory and never fails the row.

`battenCLI.wire` fills `ReviewWait` by resolving the task worktree with `taskWorktreeLocation`, building a `parentreview.Store` over its `LoomParentReviewDir`/`LoomParentReviewLockDir`, and returning `parent review: ` plus `WaitNote()`; an empty note returns empty.

`innerrun_test.go` covers the running arm with a note, without one, and with a failing `ReviewWait`.

**ImpactSummary:** One optional dep and one reason string in batten's running arm; every other arm and the routing are unchanged.
