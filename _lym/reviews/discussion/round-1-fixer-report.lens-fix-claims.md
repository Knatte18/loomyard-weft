```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked each claim the fix added or rewrote against the code at HEAD. One finding.

**Finding 1 — MEDIUM**
- **Changed text:** D5, "Its owner checks' run-scope reconcile in `gateUnlocked` and `landUnderLock`, `syncParent`'s fetched tips read and `handedOverStop`'s owner reads each stop as `git-transport` while origin is unreachable, and D6 waits them out".
- **Summary:** the sentence is wrong for `syncParent`'s fetched tips read. When that read fails today, Finalize gets a plain error with no transient class. D6 only re-steps `git-transport` and `github-api` stops, so it would not wait this one out. No decision makes Finalize mark the failure `git-transport`.
- **Evidence:**
  - `internal/landingshed/finalize.go:612-614`: `ReadParentTips(true)` returns `fmt.Errorf(...)`.
  - `internal/landingshed/finalize.go:456-457` and `internal/landingshed/stuck.go:63-66` show `gateEnding` turns that error into `fail(ctx, producer, err)` with no class.
  - Compare `handedOverStop`, which already wraps its error with `shedengine.MarkTransient(shedengine.TransientGitTransport, …)` (`internal/landingshed/finalize_owner.go:126-127`). The owner checks' reconcile gets its wrap from D6's "Finalize's owner check wraps an unreachable origin as a `git-transport` stop". Nothing similar covers `syncParent`.
- **Needed:** D5 should say Finalize marks a transport failure of `syncParent`'s fetch as `git-transport`.

**Claims that hold:**
- D1's three spawn kinds exist: `fabricengine.SpawnDetachedPush` (spawn.go:33), `SpawnStatusPush` (spawn.go:116) and `boardengine.SpawnSync` (boardengine/spawn.go:24). `SpawnPairPush` delegates to `SpawnStatusPush`, so the list covers tender's pair push.
- hubreconcile's board push is a detached `SpawnSync`, and the walk never waits on it (walk.go:218, 231-233). Dropping it from the synchronous list is correct.
- D7's hand-over move only happens when origin's tip is an ancestor of the owed tip, and a diverged tip is left to its archive tag (handover.go:259-270).
- D7's destroy move refuses when origin holds commits the checkout lacks (destroy.go:2685-2687).
- D9's merge record keeps the pre-merge `TaskHead` (finalize.go:1383), and `git check-ignore` on git 2.53.0 offers `--no-index`, `-z` and `--stdin`.

Tool calls:
1. Read `gateEnding`, hubreconcile's push lines and the spawn function names: my context showed the `gateEnding` call but not its body, and had only part of walk.go and no spawn function list.
2. Read `SpawnPairPush` and `SpawnStatusPush`: to check whether tender's `SpawnPairPush` is a fourth spawn kind D1's list misses.
