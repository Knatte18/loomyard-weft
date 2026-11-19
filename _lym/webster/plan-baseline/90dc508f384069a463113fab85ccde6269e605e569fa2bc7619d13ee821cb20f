# Card 7 — records-commit-on-code-branch

**Create:**
- `internal/fabricengine#indexLockRetryBound`
- `internal/fabricengine#retryHeldIndexLock`
- `internal/fabricengine#ErrPrimeBranch`
- `internal/fabricengine#IsPrimeBranchRefusal`
- `internal/fabricengine#refuseOffPrimeBranch`
- `internal/fabricengine#TestRetryHeldIndexLock`
- `internal/fabricengine_test#TestRecordsCommit_OneRepository`

**Edit:**
- `internal/fabricengine#commitRecordsPathsHeld`
- `internal/fabricengine#CommitEmptyWithTrailers`
- `internal/fabricengine#Fabric.commitWeftLocked`
- `internal/fabricengine#Fabric.commitEmptySnapshot`
- `internal/fabricengine#takeoverBranch.commitEntry`
- `internal/fabricengine#migrateWeftMarker`
- `internal/fabricengine#recordAdoptedWeftAnchor`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#PrimeBranchConfigKey`
- `internal/fabricengine#oneRepositoryAt`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#NewHub`
- `internal/hubforge#AddPairWith`

**Intent:** D4's records commit on the worktree's own branch, and D7's prime-branch refusal.
With card 3's geometry every records commit (`lym fabric commit`, `sync`, the transition commit, `webstercli`'s sync, a merge's step 1) already targets the worktree; this card makes it safe there.

- Path-limited: `commitRecordsPathsHeld` and `commitWeftLocked` keep `gitrepo.StageAndCommit`, whose `git commit -- <paths>` commits only the scoped paths and leaves other staged entries staged.
  The paths stay `ScopedPathspec` results, so code an agent staged is never swept in.
- `commitEmptySnapshot` in a one-repository worktree commits through `CommitPathAllowEmpty` over `.lym-anchor` in place of `CommitEmpty`,
  which refuses a non-empty index; the agent's staged entries stay staged and none is committed.
- One write lock: `Fabric.commitBothSides` already takes the write lock for its code and records commits, and card 3 puts that lock in the worktree's gitdir, which every records committer takes, so the two run one after the other.
  `TestRecordsGeometry_OneRepository` pins the shared lock path.
- `retryHeldIndexLock` runs `commit`; while the error names a held `index.lock` (git's `index.lock': File exists`) it sleeps, doubling from 50ms, and runs it again until the slept time reaches `bound`, then returns the last error, which fails as a commit failure does today.
  Any other error returns at once. It sums the slept time and reads no clock, so a test drives it with a fake `sleep`.
  `commitRecordsPathsHeld`, `commitWeftLocked` and `CommitEmptyWithTrailers` wrap their git commit in it with `indexLockRetryBound` and `time.Sleep`.
- `Warp-SHA`: `commitWeftLocked` and `messageWithTrailers` read HEAD before the records commit, which in one repository names HEAD before the commit; `Run-Transition` is unchanged.
- `refuseOffPrimeBranch(worktree)` applies only when `oneRepositoryAt(worktree)` holds and `worktree` is that repository's main worktree, the prime;
  a two-repository records worktree, the weft prime included, is never refused.
  It reads `lym.primeBranch` (`PrimeBranchConfigKey`) and the checked-out branch, and refuses with `*ErrPrimeBranch`:
  an absent key refuses naming the key, way forward: run `git config lym.primeBranch <branch>` in the prime, naming the branch its records live on, then run the step again;
  a different branch refuses naming both branches, way forward: switch the prime back to the recorded branch, with `git switch --merge` when git refuses to carry the uncommitted records.
  `IsPrimeBranchRefusal` reads the error through `errors.As`; card 11's seam waits on it.
  `commitRecordsPathsHeld`, `CommitEmptyWithTrailers` and `commitWeftLocked` call it before they stage anything, so the records stay uncommitted on disk,
  and every records commit, a merge's step 1 and `MergeAbort`'s included, refuses there.
  Bound: while the prime is off its recorded branch every prime run's records commit refuses, so prime runs stall until the switch back.
  The key lets records commits land on whichever branch it names while the prime has that branch checked out, so an operator who sets it to another branch moves the prime's records there.
  It binds only the main worktree of a one-repository hub, never a task worktree or a two-repository records worktree, and every records push it leads to still passes card 14's guard.
- Records commits outside the three primitives take the same refusal and retry:
  `takeoverBranch.commitEntry` runs `refuseOffPrimeBranch` before it writes the takeover entry and wraps its `StageAndCommit` in `retryHeldIndexLock`, as the primitives do.
  `migrateWeftMarker` and `recordAdoptedWeftAnchor`, the two that commit a missing marker through `commitAnchorMarker` on an existing worktree, run `refuseOffPrimeBranch` first
  and wrap the `commitAnchorMarker` call in `retryHeldIndexLock`; a held-lock failure commits nothing, so the re-run commits the marker once.
  `commitAnchorMarker` itself stays as it is, since card 5's clone runs it on a freshly cloned prime with no git operation in progress.
  Off its recorded branch the prime's marker commit now refuses: `migrateWeftMarker` reports the refusal in its pair's `Error`, and `recordAdoptedWeftAnchor` returns it.
  On a recorded branch lacking the marker, which `lym.primeBranch` can name, the commit lands there, and card 10 refuses it while a git operation is in progress.

Tests:
- `TestRetryHeldIndexLock` (tier0, package `fabricengine`) rows: `ReleasedWithinTheBoundLands` (two held-lock errors, then nil; sleeps 50ms and 100ms); `HeldPastTheBoundFails` (always held: returns the held-lock error once the sleeps sum to the bound); `OtherErrorIsNotRetried` (no sleep).
- `TestRecordsCommit_OneRepository` (tier1, `NewOneRepoHub`, one pair through `AddPairWith`) rows:
  `CommitsOnlyRecordsAndLeavesStagedCodeStaged` (a code file staged by `git add` in the pair; `CommitAnchoredPathsWithTrailers` of a status file commits only that path, the code stays staged, and `Warp-SHA` names the HEAD before the commit);
  `EmptyRecordsCommitWithCodeStagedLands` (`Fabric.Commit(nil, msg, tags, ...)` lands an empty commit carrying the tags with the code still staged);
  `PrimeOffItsBranchRefuses` (prime switched to another branch with a records edit: the error is `*ErrPrimeBranch` naming both branches, `IsPrimeBranchRefusal` reads it, the edit stays on disk uncommitted);
  `PrimeWithoutTheKeyRefuses` (`lym.primeBranch` unset: refusal names the key);
  `TwoRepositoryPrimeIsNotRefused` (a `NewHub` hub's prime records commit lands with no `lym.primeBranch` key).

**ImpactSummary:** Every records commit in fabricengine gains the index.lock retry and, in a one-repository prime, the branch refusal; two-repository commits differ only by the retry.
