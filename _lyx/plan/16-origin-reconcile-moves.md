# Card 16 — origin-reconcile-moves

**Create:**
- `internal/fabricengine#ffWorktree`
- `internal/fabricengine#reconcileRecordsBranch`
- `internal/fabricengine#tryPrimeLock`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`

**Edit:**
- `internal/fabricengine#ReconcileOrigin`

**Uses:**
- `internal/fabricengine#readEndedSet`
- `internal/fabricengine#finalizeEntered`
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#replayBolt`
- `internal/fabricengine#classifyEntry`
- `internal/fabricengine#SetOriginCallHookForTest`
- `internal/testkit/boardkit#InstallReplay`

**Intent:** D3's order of work and D4's moves: the moving reconcile.
After the fetches `ReconcileOrigin` reads the ended set from the fetched origin Bolt tip and its history (`readEndedSet`, over the slugs this machine holds a batten run record of and those a local records commit origin lacks touches), before any replay, so it holds whatever local Bolt did before.
Right after `readEndedSet`, before Bolt's replay and any move, it calls `finalizeEntered` once for every candidate slug and drops each entered one, and passes that filtered `EndedSet` to every replay and to card 17's `reconcileEnded`, so D6's judgment reads the local parent before the reconcile fast-forwards it.
It then brings Bolt level (fast-forward, or `reconcileRecordsBranch`'s replay), then the prime's branches, the prime weft before the prime's warp branch, so Bolt's and the prime weft's replays' own `finalizeEntered` drop also reads the parent unmoved, then each scoped pair's, whose weft replay's own drop may read a moved parent but only removes slugs from the filtered set, keeping commits and never setting aside this machine's landing, per D4:
- A branch checked out in a worktree moves only by `ffWorktree`, a fast-forward of that worktree; a tracked change on a path the move touches is **dirty**, naming the paths, with nothing moved.
- The prime's branches move only under the prime lock taken by `tryPrimeLock`, one try-acquire with no wait, and only when `OriginSeams.PrimeParentLockFree` finds no live owner of the prime branch's parent lock record; otherwise the prime is **busy**, judged and not moved. The reconcile takes no parent lock ticket.
- The run scope never moves the parent branch: it is judged only, for Merge-In's own fast-forward.
- A pair whose `OriginSeams.LivePair` reports live, outside the run scope's own pair, is judged and reported **live**, never moved; the seam reads a pair a live hub pause entry covers as not live. The run scope never asks `LivePair` about its own pair, whose driver is the caller.
- A records branch moves under its write lock, then its push lock (on Bolt `board.lock` then `.gitrepo-push.lock`; on a weft the weft's write lock then its push lock), each by one try-acquire: a held lock is **busy**, judged and not moved. `reconcileRecordsBranch` fast-forwards, or calls `replayWeft`/`replayBolt` with the ended set; a replay stop is **diverged** naming the conflicting paths, and dirtiness is **dirty**.
A code branch is never rebased: diverged at once.
Each move fills the `BranchOutcome.WayForward` card 15 lists for the state it reports: dirty, busy, diverged and live.
A stop on Bolt or on the prime's records branch stops every scoped entry, each `stopped` with that cause (D3); a busy Bolt or prime weft is no such stop, and a busy run-scope own weft stops nothing; a busy pair weft stops its entry, outside the run scope's own pair.
A judge-only scope runs none of this: no move, no replay.
Bound: the reconcile moves a local branch only forward to origin's head or by D5's replay; it never resets away a commit origin lacks, never force-pushes, never deletes or recreates a remote branch, never waits on the prime lock, a parent lock, a records write lock or a push lock, runs no store gc, and never moves a branch a live run, a landing or a held lock is using.

`TestReconcileOrigin_Moves` (integration, `hubforge` hub, second clones as the other machine, seams faked, `boardkit.InstallReplay` called first, run without `t.Parallel` since it sets the call hook) rows: origin moved on a weft, Bolt and warp `main` with nothing local only, a hub reconcile fast-forwards all three and reports each, the report's `Mutations.Entries()` holding one `repo_advanced` entry per fast-forward, and the same origin moves under a judge-only scope leave `Mutations.Entries()` empty, the call hook counting exactly two ls-remote calls and fetches naming only the moved heads; a pair worktree whose weft origin moved, clean on the touched paths, is fast-forwarded, and a tracked change on a touched path stops it as dirty with nothing moved, its `WayForward` exactly the pair weft's dirty line, `lyx fabric commit` in the pair's task worktree, with no re-entry verb; a tracked change on a path the prime weft's fast-forward touches is dirty with `WayForward` exactly the prime weft's line, `lyx fabric commit` in the prime worktree; a tracked change on a path a pair's code branch fast-forward touches is dirty with `WayForward` exactly the code line, `git commit` or `git checkout -- <path>` in the pair's task worktree; a live pair is reported live and not moved, its entry `active` while level or ahead and `stopped` once origin moved past it, its `WayForward` then the live line exactly; under run scope the run's own pair, whose `LivePair` fake reads live as its driver holds the run lock during Merge-In, is moved like a quiet pair and its entry stays `active`; the prime with its prime lock held, and with a live parent lock owner, is busy at once and not moved; run scope with the parent behind origin judges and does not move it; a pair weft replay that stops maps to a `diverged` branch naming the replay's conflicting paths, `Summary` naming `recordsConflictWayForward`'s exact line, its git steps in the weft worktree and `lyx fabric push` in the pair's code worktree, and stops that entry (the replay's own behaviour is card 4's); a diverged code branch stops and is not rebased; a Bolt replay under the reconcile's `board.lock` and `.gitrepo-push.lock` completes without waiting and spawns no sync, with the call hook still counting two ls-remote calls, and, holding a local status commit of an entry done on origin, sets it aside with the report's `Mutations.Entries()` holding one `tag_created` entry per set-aside tag; a tracked change on a path the Bolt replay touches stops every scoped entry, Bolt's `WayForward` the dirty line's Bolt form exactly, and a both-sides change of the prime weft's records does the same, its `WayForward` `recordsConflictWayForward`'s exact line with `lyx fabric push` in the prime's worktree; a held pair weft write lock is busy and stops that entry, its `WayForward` the busy write-lock line exactly, with no re-entry verb; a held pair weft push lock is busy with `PushLockHeld` set and stops that entry; a held Bolt push lock, a held prime weft write lock and, under run scope, a held write lock of the run's own weft each report busy and stop nothing.

**ImpactSummary:** Extends the one new entry point only; no caller exists until cards 19 to 23.

**Commit:** 16: fabricengine: the origin reconcile moves what no live writer meets
