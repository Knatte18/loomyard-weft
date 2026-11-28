# Card 4 — fabric-detached-checkout

**Create:**
- `plan:internal/lymdirs#DetachMarkerName` -> `const DetachMarkerName = "detach-marker"`
- `plan:internal/fabricengine#DetachMarkerPath` -> `func DetachMarkerPath(l *lymcwd.Location) string`
- `plan:internal/fabricengine#WriteDetachMarker` -> `func WriteDetachMarker(l *lymcwd.Location, branch string) error`
- `plan:internal/fabricengine#ReadDetachMarker` -> `func ReadDetachMarker(l *lymcwd.Location) (string, bool, error)`
- `plan:internal/fabricengine#RemoveDetachMarker` -> `func RemoveDetachMarker(l *lymcwd.Location) error`
- `plan:internal/fabricengine#Fabric.CheckoutDetached` -> `func (f *Fabric) CheckoutDetached(commit string) error`
- `plan:internal/fabricengine#Fabric.CheckoutBranch` -> `func (f *Fabric) CheckoutBranch(branch string) error`
- `plan:internal/fabricengine#DetachState` -> `type DetachState struct`
- `plan:internal/fabricengine#ReadDetachState` -> `func ReadDetachState(l *lymcwd.Location) (DetachState, error)`
- `plan:internal/fabricengine#RestoreDetached` -> `func RestoreDetached(l *lymcwd.Location) (DetachState, error)`
- `plan:internal/fabricengine#detachWayForward` -> `func detachWayForward(l *lymcwd.Location) string`
- `internal/fabricengine/detach_integration_test.go`
- `plan:internal/fabricengine_test#TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` -> `func TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair(t *testing.T)`

**Edit:**
- `internal/fabricengine#readLocalBranch`
- `internal/fabricengine#RefusedPushStands`
- `internal/fabricengine#pushClassified`
- `internal/fabricengine#parentHoldsTaskWorkAt`
- `internal/fabricengine_test#TestRunCommitsAhead`
- `internal/fabricengine_test#TestRefusedPushStands`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/lymdirs/doc.go`
- `internal/fabricengine#RemoveIgnoredLeftovers`
- `internal/fabricengine#RunCommitsAhead`
- `internal/fabricengine#TipsOnOrigin`
- `internal/fabricengine#PushPairAnchored`
- `internal/runpusher#targetAhead`
- `internal/runpusher#TargetRefused`

**Intent:** Add the detached checkout of D5 and its restore as fabric verbs, the marker that says a detach is lym's own, and the reads that judge a detached task worktree by its task branch ref.

Marker.
`lymdirs.DetachMarkerName` names a file under the worktree's own `.lym`; `DetachMarkerPath(l)` joins it under `l.WorktreePath()` with `lymdirs.DotLymDirName`.
The file holds the task branch's name and a newline; `WriteDetachMarker` writes it atomically, `ReadDetachMarker` returns the name and whether it stands, and `RemoveDetachMarker` removes it, an absent one being no error.
It is a cache (`PATTERN-lym-files`): an absent marker reads as none.
`detachWayForward(l)` words the one way forward every error below about a marker gives: "check the task branch out in <worktree> with plain git, remove <marker path>, then run `lym loom resume` in the task worktree".

Checkouts, on the pair's code checkout alone, never the records side:
- `CheckoutDetached(commit)` refuses a checkout with tracked or untracked dirt, naming the paths, then runs `git checkout --detach <commit>` through `gitrepo`.
- `CheckoutBranch(branch)` refuses the same dirt, then runs `git checkout <branch>`; it never moves the branch ref.
- `DetachState` holds `Detached`, `Commit` (the detached head), `Marker` (whether one stands), `Branch` (the marker's), `Dirty []string` and `Restored`.
  `ReadDetachState(l)` fills all but `Restored`.
- `RestoreDetached(l)` reads the state.
  A worktree on a branch removes a stale marker and returns.
  A detached clean worktree with a marker is checked out on the marker's branch, at whatever commit it stood; `RemoveIgnoredLeftovers(l, commit)` then removes the ignored files left in directories the checkout deleted since that commit; the marker is removed; `Restored` is set.
  A marker naming a branch that does not exist is an error naming the branch and the marker, ending in `detachWayForward`, and moves nothing.
  A detached dirty worktree, and a detached one with no marker, are returned as they stand with nothing moved, for the caller to stop on (cards 8 and 10 word those stops).

Reads while a marker stands, so a detached worktree is judged by its task branch, never its HEAD (D5):
- `readLocalBranch(path)` reads the tip of `refs/heads/<marker branch>` and that branch's upstream when a marker stands at `path`'s `.lym`, and HEAD's branch otherwise.
  Its every caller so reads the task branch: `RunCommitsAhead` (through which `runpusher`'s `targetAhead` reads a pair), `TipsOnOrigin`, whose hand-over and takeover reads then judge the task branch, and `PushPairAnchored`'s own read at `pushanchored.go:132`.
- `RefusedPushStands` takes the code branch from the marker when one stands, since `CurrentBranch` fails on a detached HEAD.
  `runpusher`'s `TargetRefused` reads it unchanged.
- `pushClassified` on the code side of a worktree whose marker stands pushes nothing and returns a `*PushFailedError` of the `deferred` class (`fabriclayout.PushFailureDeferred`), whose text names the marker and `detachWayForward`, so the pusher waits it out as an owed push rather than reading it as pushed and spinning, and `RefusedPushStands` never counts it as a refusal.
  Its callers (`PushPairAnchored`, `coalesce.go`'s side push, `bolt.go`'s and `landing.go`'s pushes) already report a deferred push as owed; the Bolt and prime pushes never reach the code side of a task worktree.
  Bound: only the code side of the marked worktree is held off, and only while the marker stands; the records side pushes as before.
  The hold ends when the leader's last restore removes the marker, or, after a leader's death, at the run's `lym loom resume`, whose shed restore removes it (card 10).
- `parentHoldsTaskWorkAt` reads the task head from the marker's branch ref when a marker stands, and from the code checkout's head otherwise, so `ParentHoldsTaskWork` resolves the task branch.
  A marker that names no existing branch, as after the branch was deleted by hand, is an error naming the marker and ending in `detachWayForward`.

`detach_integration_test.go` is tier1, in `package fabricengine_test`, over a `hubforge` hub with one task pair.
`TestRestoreDetached` rows: a clean worktree detached at an earlier commit with a marker is back on its branch, the branch ref unchanged and the marker gone; one detached at a commit whose directory the branch deleted, holding ignored build output there, is restored clean; a dirty detached one returns its dirty paths and stays detached with the marker kept; a detached one with no marker returns its commit and stays detached; a worktree on its branch with a stale marker loses the marker; a marker naming a deleted branch errors with the marker path and `lym loom resume` in its text and stays detached.
`TestCheckoutDetached` rows: a clean checkout detaches at the commit; a dirty one is refused naming the path and stays on its branch; `CheckoutBranch` over a dirty detached checkout is refused naming the path and stays detached.
`TestParentHoldsTaskWork_ReadsTheBranchWhileDetached` rows: a worktree detached at a commit the parent holds, with a marker naming a branch the parent does not hold, reads false; a marker naming a deleted branch errors naming the marker.
`TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` pins that a pair detached with its marker standing, its records branch ahead, pushes its records side, pushes nothing on the code side, and reports the code side as a deferred push naming the marker; after the marker is removed and the branch checked out the next push sends the code side.
`TestRunCommitsAhead` gains a row: a pair detached at a commit origin lacks, with its marker standing and its branch level with origin, reads not ahead.
`TestRefusedPushStands` gains a row: a refused-push record of the code branch stands for a pair detached with its marker naming that branch.

**ImpactSummary:** Four fabric reads and the code-side push change for a marked worktree only; an unmarked worktree reads and pushes as before, and every caller already handles a deferred push.
