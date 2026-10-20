# Card 16 — origin-reconcile-moves

**Create:**
- `plan:internal/fabricengine#ffWorktree` -> `func ffWorktree(rec *Mutations, worktree, originHead string) (state BranchState, dirty []string, err error)`
- `plan:internal/fabricengine#reconcileRecordsBranch` -> `func reconcileRecordsBranch(rec *Mutations, l *lyxcwd.Location, worktree, originHead string, ended EndedSet, bolt bool) (BranchOutcome, error)`
- `plan:internal/fabricengine#tryPrimeLock` -> `func tryPrimeLock(l *lyxcwd.Location) (release func(), held bool, err error)`
- `plan:internal/fabricengine_test#TestReconcileOrigin_Moves` -> `func TestReconcileOrigin_Moves(t *testing.T)`

**Edit:**
- `plan:internal/fabricengine#ReconcileOrigin`

**Uses:**
- `plan:internal/fabricengine#readEndedSet`
- `plan:internal/fabricengine#replayWeft`
- `plan:internal/fabricengine#replayBolt`
- `plan:internal/fabricengine#classifyEntry`
- `plan:internal/fabricengine#SetOriginCallHookForTest`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D3's order of work and D4's moves: the moving reconcile.
After the fetches `ReconcileOrigin` reads the ended set from the fetched origin Bolt tip and its history (`readEndedSet`, over the slugs this machine holds a batten run record of and those a local records commit origin lacks touches), before any replay, so it holds whatever local Bolt did before.
It then brings Bolt level (fast-forward, or `reconcileRecordsBranch`'s replay), then the prime's branches, then each scoped pair's, per D4:
- A branch checked out in a worktree moves only by `ffWorktree`, a fast-forward of that worktree; a tracked change on a path the move touches is **dirty**, naming the paths, with nothing moved.
- The prime's branches move only under the prime lock taken by `tryPrimeLock`, one try-acquire with no wait, and only when `OriginSeams.PrimeParentLockFree` finds no live owner of the prime branch's parent lock record; otherwise the prime is **busy**, judged and not moved. The reconcile takes no parent lock ticket.
- The run scope never moves the parent branch: it is judged only, for Merge-In's own fast-forward.
- A pair whose `OriginSeams.LivePair` reports live, outside the run scope's own pair, is judged and reported **live**, never moved; the seam reads a pair a live hub pause entry covers as not live. The run scope never asks `LivePair` about its own pair, whose driver is the caller.
- A records branch moves under its write lock, then its push lock (on Bolt `board.lock` then `.gitrepo-push.lock`; on a weft the weft's write lock then its push lock), each by one try-acquire: a held lock is **busy**, judged and not moved. `reconcileRecordsBranch` fast-forwards, or calls `replayWeft`/`replayBolt` with the ended set; a replay stop is **diverged** naming the conflicting paths, and dirtiness is **dirty**.
A code branch is never rebased: diverged at once.
A stop on Bolt or on the prime's records branch stops every scoped entry, each `stopped` with that cause (D3); a busy Bolt or prime weft is no such stop, and a busy run-scope own weft stops nothing; a busy pair weft stops its entry, outside the run scope's own pair.
A judge-only scope runs none of this: no move, no replay.
Bound: the reconcile moves a local branch only forward to origin's head or by D5's replay; it never resets away a commit origin lacks, never force-pushes, never deletes or recreates a remote branch, never waits on the prime lock, a parent lock, a records write lock or a push lock, runs no store gc, and never moves a branch a live run, a landing or a held lock is using.

`TestReconcileOrigin_Moves` (integration, `hubforge` hub, second clones as the other machine, seams faked, `boardkit.InstallReplay` called first, run without `t.Parallel` since it sets the call hook) rows: origin moved on a weft, Bolt and warp `main` with nothing local only, a hub reconcile fast-forwards all three and reports each, the call hook counting exactly two ls-remote calls and fetches naming only the moved heads; a pair worktree whose weft origin moved, clean on the touched paths, is fast-forwarded, and a tracked change on a touched path stops it as dirty with nothing moved; a live pair is reported live and not moved, its entry `active` while level or ahead and `stopped` once origin moved past it; under run scope the run's own pair, whose `LivePair` fake reads live as its driver holds the run lock during Merge-In, is moved like a quiet pair and its entry stays `active`; the prime with its prime lock held, and with a live parent lock owner, is busy at once and not moved; run scope with the parent behind origin judges and does not move it; a pair weft replay that stops maps to a `diverged` branch naming the replay's conflicting paths and stops that entry (the replay's own behaviour is card 4's); a diverged code branch stops and is not rebased; a Bolt replay under the reconcile's `board.lock` and `.gitrepo-push.lock` completes without waiting and spawns no sync, with the call hook still counting two ls-remote calls; a tracked change on a path the Bolt replay touches stops every scoped entry, and a both-sides change of the prime weft's records does the same; a held pair weft write lock is busy and stops that entry; a held pair weft push lock is busy with `PushLockHeld` set and stops that entry; a held Bolt push lock, a held prime weft write lock and, under run scope, a held write lock of the run's own weft each report busy and stop nothing.

**ImpactSummary:** Extends the one new entry point only; no caller exists until cards 19 to 23.

**Commit:** 16: fabricengine: the origin reconcile moves what no live writer meets
