---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/add.go:852
    summary: Add rollback now finishes a deregistered or refilled weft directory but no test drives a failed Add with such a directory
    fix: In internal/fabricengine/remove_leftoverdir_integration_test.go, or beside the rollback tests in add_rollback_adopt_test.go, add one case that forces Add to fail after the weft worktree exists, sets a refill through SetLeftoverRefillForTest on the weft path (the rollback's git remove exits cleanly), and asserts that Add's error is returned and the weft directory is gone afterwards. Reuse the failure injection the existing rollback tests already use rather than a new one.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/destroy.go:1409
    summary: The registered-worktree guard in finishGitWorktreeRemoval is the data-loss safeguard of the sibling side and no test fails if it is dropped
    fix: Add a row to TestRemove_FinishesWorktreeDirGitLeftOnDisk, or a small sibling test, where git exits nonzero on a sibling that stays registered (for example a worktree locked with git worktree lock, or a dirty sibling with force false that survives the records-disposable gate) and assert that Remove returns the git error, that the sibling directory and its content stay on disk, and that no path_removed mutation is recorded for it.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/destroy.go:1417
    summary: The path where git deregistered the sibling, failed, and the finish also fails has no row on the sibling side
    fix: Add a row to TestRemove_FinishesWorktreeDirGitLeftOnDisk with sibling true and readOnly true and a removeErr set, and assert that Remove returns an error wrapping removeErr, naming that git deregistered the directory, and that the sibling directory is still on disk. This covers the gitErr non-nil wrapper of finishGitWorktreeRemoval and the weft worktree remains outcome of removeSibling.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go:1410
    summary: A failed worktree listing reads as not registered and so licenses a recursive delete of a worktree git refused to remove
    fix: In finishGitWorktreeRemoval, decide registration from the List result itself rather than from isRegisteredLinkedWorktreeIn, which answers false on an enumeration error. When listing the repository's worktrees fails, return gitErr unchanged. Keep isRegisteredLinkedWorktreeIn as it is for its other callers, and add the failure case to the existing predicate test or the helper's row set only if a cheap fake exists.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go:1421
    summary: When the finish fails after a clean git exit the error gives no way forward and a retry of Remove reports the directory as stray and never deletes it
    fix: Extend the clean-exit error message in finishGitWorktreeRemoval to name the way forward, that the directory is no longer a worktree and can be deleted by hand once whatever refills it is stopped, then Remove run again. Keep the wrapped cause with percent-w.
  - id: F6
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:9
    summary: The summary states the late-writer cause as established and the single check straight after git exits as closing it, but the diff carries no evidence of the cause and a refill after that check still leaves the directory
    fix: In _lym/landing/summary.md, word the Cause section as the cause the change addresses, not a finding, and add that the directory is checked once after git exits, so a writer that refills later is not covered.
---

### [MEDIUM] Add rollback leftover finish has no test

**Location:** internal/fabricengine/add.go:852
**Issue:** The diff passes `t.leftover` into `removeWeftWorktree` from `rollbackAdd`, so a failed Add now deletes a weft directory a late writer refilled, and the summary says so. The only new test calls `Topology.Remove` (`remove_leftoverdir_integration_test.go:113`). The existing rollback tests (`add_rollback_adopt_test.go:120`, `origin_integration_test.go:269`) never set the refill hook, so removing the argument would break none of them.

### [MEDIUM] Registered-worktree guard of the sibling side is unpinned

**Location:** internal/fabricengine/destroy.go:1409
**Issue:** The guard `isRegisteredLinkedWorktreeIn` keeps `finishGitWorktreeRemoval` from deleting a sibling directory git refused to remove while it was still registered. It is the only thing between a git refusal and a recursive delete on the sibling side, which this diff newly opens. The sibling row reaches only the deregistered case. A change that drops the guard fails no test in the diff; the task-side tests reach `removeWarpWorktreeDir`'s own branch, not this helper's.

### [LOW] Sibling finish failure after a git failure is untested

**Location:** internal/fabricengine/destroy.go:1417
**Issue:** The `gitErr != nil` branch that wraps both errors, and the "weft worktree remains" result in `removeSibling` and `removeLocalWeft`, have no row. The failing-removal row covers only the task side after a clean exit.

### [LOW] Listing failure licenses a recursive delete

**Location:** internal/fabricengine/destroy.go:1410
**Issue:** `isRegisteredLinkedWorktreeIn` returns false when `List` fails (`remove.go:1106`), and its doc calls false the conservative direction. In `finishGitWorktreeRemoval` false means deregistered, so a git refusal on a registered, dirty sibling followed by a failed listing deletes the worktree, including with force false. The task side already has this shape in `removeWarpWorktreeDir`; this diff extends it to the sibling side. Likelihood is low.

### [LOW] No way forward after a failed finish

**Location:** internal/fabricengine/destroy.go:1421
**Issue:** Before this diff a refilled directory after a clean git exit let Remove report success. Now Remove returns an error with the task worktree deregistered, and a retry classifies the directory as a stray path (`inspectPair`, `remove.go:580-585`), which Remove reports and never deletes. The message names no step the operator can take. The nonzero-exit path shares the limit, so this is not a regression of that path.

### [NIT] Summary states an unverified cause and a closed window

**Location:** _lym/landing/summary.md:9
**Issue:** The Cause section presents the late writer as established, and the Change section as closing the case. The diff holds only a simulated refill, and `finishGitWorktreeRemoval` looks once immediately after git exits, so a writer that refills after that look is not covered.

## Rejected

- Route the nonzero-exit branch of `removeWarpWorktreeDir` through `finishGitWorktreeRemoval` (lens:code-affected-set, LOW consistency): the split is disclosed in the summary and commit message, the two paths use the same ownership predicate with the same container, and the task-side error wording is pinned by `destructivegaps_integration_test.go:359`.
- Containment of the sibling and task targets under `l.HubPath`: `RecordsWorktreePath` and `WorktreePath` both resolve to hub children, so `removeContainedPath` does not refuse them.
- Doc comments of `finishGitWorktreeRemoval` and the changed callers naming other symbols: the surrounding code in `destroy.go` and `remove.go` documents the same way, and line breaks are the comment lint's.
- `doc.go` paragraph and `PATTERN-fabric-git.md` exception for `removeWorktreeLeftover`: both stay true, since the clean-exit path also follows a gated `git worktree remove` on the same target, and the directive declares no PATTERN change.
- Test economy of `remove_leftoverdir_integration_test.go`: each row reaches a distinct caller or git outcome, and the only overlapping existing test (`destructivegaps_integration_test.go`, task-side read-only rows) does not reach the sibling side. The read-only row skips for root with a stated prerequisite, is not generated on Windows, and restores permissions in `t.Cleanup`.
