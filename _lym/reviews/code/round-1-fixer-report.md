# Fixer-report, code review round 1

All six findings fixed. The fabricengine tier-1 gate passes (1419 passed, 0 failed).
The fixes went in as one commit (`fabricengine: fix F1-F6 ...`), not one per finding, because F1, F3, F4 and F5 rewrite the same functions and doc comments.
A second commit holds the self-check fixes.
A third commit repairs the gate's comment findings: three comments in `add_taskpathleftover_integration_test.go` were wrapped mid-sentence or over 120 characters, and are now one sentence per line within the limit.

## Changes

- **F1** (fixed as proposed, delete moved): `Add` judges the task path at the original position and returns `*ErrTaskPathTaken` if it is not deletable.
  It deletes only just before `createGitWorktree`, after `defer regenerateAfterVerb` and the pre-flight refusals.
  `removeIgnoredTaskPath` is split into the read-only `ignoredTaskPathDeletable` and the delete step `removeIgnoredTaskPath`.
  The `Add` doc comment states that a refused `Add` leaves the directory.
  Test: `TestAdd_TaskPathLeftoverKeptWhenPreflightRefuses`, an adopt-only add refused with `ErrNotLive` that must leave the file.
- **F2** (fixed as proposed): row `TrackedFileKeepsIt` in `TestAdd_TaskPathLeftover` commits `src/App/obj/tracked.cs` before the ignore rule and expects `*ErrTaskPathTaken` with every file kept.
  No registered-worktree row was added, as the review advised.
- **F3** (fixed as proposed): the `ErrTaskPathTaken` type comment and `Error` text now say the refusal is for a path Add does not delete.
- **F4** (fixed as proposed): the failed delete wrapping `ErrStrayPathKept` is stated in the `Add` doc comment, in `doc.go` (the Add paragraph and the sources bullet) and in the `ErrStrayPathKept` comment in `remove.go`.
- **F5** (fixed as proposed): a failed `Lstat`, `List` or walk logs a warning, keeps the path and reports not deletable with no error, so `Add` returns `*ErrTaskPathTaken`.
- **F6** (fixed as proposed): row `NothingIgnoredKeepsIt` (check-ignore exit 1), and `TestAdd_TaskPathLeftoverDeleteFails`, which fails the delete through `SetLeftoverSeamForTest` and asserts `ErrStrayPathKept` and the file on disk.
- `doc.go` also notes that `ignoredTaskPathDeletable` judges the proof in the pre-flight.

## Self-check

- fix-code-correctness, LOW: `ErrTaskPathTaken` comment said an ignored-only directory is never refused. Fixed as proposed, now "unless the path cannot be judged".
- fix-code-correctness, LOW: the keep warning said git could not judge, although the cause can be `Lstat` or `List`. Fixed as proposed, the message is now neutral.
- fix-consistency: no findings.
- fix-proposal, MEDIUM: the time between judging and deleting spans the origin probes, so a file saved there meanwhile would be deleted. Fixed with my own change: `Add` judges the path again just before the delete and refuses with `*ErrTaskPathTaken` if it is no longer deletable. The reviewer's split alone did not close the gap.
- fix-proposal, LOW: keep warning wording. Same fix as above.
- fix-proposal, LOW: the `ErrTaskPathTaken` message was false for a live worktree and a non-directory. Fixed as proposed, reworded to "fabric does not delete it, since fabric deletes only a directory of git-ignored output".
- fix-affected-set, LOW: `battencli` `teardownRefusal` says "the pair was removed". No Add path reaches it, so nothing was changed.
- fix-affected-set, LOW: `pairteardown` docs describe the sentinel for teardown only. They are not sources of it, so nothing was changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
