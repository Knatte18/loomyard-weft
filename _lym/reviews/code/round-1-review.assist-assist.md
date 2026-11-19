---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/onerepository.go:42
    summary: Cached layout survives worktree removal, replacement and resets across marker presence.
    fix: "In onerepository.go, validate cached worktree identity and existence before returning an answer. Invalidate affected entries at fabric's worktree removal, replacement and HEAD-changing boundaries, including destroy.go's removeGitWorktree and resetHardTo and clone.go's resetHub. Keep repeated reads of unchanged worktrees free of git commands. Extend public records-geometry coverage with cached-read, removal, missing-read and recreation cases, plus a linked worktree reset across marker presence. Update the landing summary's claim that marker writers are the only relevant mutations."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/onerepository_quiet_test.go:36
    summary: The quiet-layout regression tests a private helper instead of the public geometry surface.
    fix: "Change the existing quiet-layout test to call RecordsWorktreePath with a synthetic lymcwd.Location rooted at each fixture. Assert the expected records sibling path and retain the empty-log assertion. Update its name and comments to describe the public behavior, and use the external test package if no private symbols remain."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/onerepository.go:54
    summary: An in-flight reader can repopulate the cache with an obsolete answer after invalidation.
    fix: "Coordinate cache fills and forgetLayouts with shared synchronization or a generation checked atomically with publication. A read started before invalidation must not publish its old answer afterward. Retry invalidated reads and add a deterministic interleaving case for a marker write between layout computation and cache publication."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/mergestateactive_integration_test.go:97
    summary: The restored two-repository revert and rebase dispositions lack regression rows.
    fix: "Extend TestMergeStateActive's existing scenario coverage with clean-index states for REVERT_HEAD, rebase-merge and rebase-apply, with no MERGE_HEAD. Assert false from MergeStateActive and an empty MergeStateOperation for each, cleaning the operation state between cases. Preserve the existing conflicted-index check."
---

### [BLOCKING] Cached answers outlive their worktree state

**Location:** internal/fabricengine/onerepository.go:42

**Issue:** The cache returns before checking whether the worktree still exists.
After a true answer is cached, removing the worktree leaves subsequent reads returning true.
`RecordsWorktree` then returns the deleted code path instead of the missing-path fallback required by the Goal.
`removeGitWorktree` at `destroy.go:1536` and `resetHub` at `clone.go:644` do not invalidate the cache.
`resetHardTo` at `destroy.go:1938` also leaves it intact after replacing HEAD.
A linked worktree reset to a commit without the marker therefore retains its old classification.
That contradicts the marker-based layout rule in `pattern/PATTERN-single-repo-hub.md`.
Reusing the path for another repository can preserve the obsolete answer too.
The correctness lens independently reported the removal and replacement case; it is consolidated here.
The landing summary's claim that only the listed marker writers change the probe's answer does not hold.

### [BLOCKING] Quiet-layout coverage bypasses the public surface

**Location:** internal/fabricengine/onerepository_quiet_test.go:36

**Issue:** `PATTERN-test-speed` requires testing an unexported helper through its public surface.
The new test calls `oneRepositoryAt` directly.
`RecordsWorktreePath` at `weftwiring.go:36` exposes the same quiet missing-path behavior without extra git work.
Its returned sibling path and captured log output provide the required observable assertions.
The private helper's visibility does not justify bypassing that surface.

### [MEDIUM] Invalidation does not exclude stale publication

**Location:** internal/fabricengine/onerepository.go:54

**Issue:** Reading the layout, storing it and clearing the cache are independent operations.
A reader can compute false before a marker commit, then pause.
The writer commits and calls `forgetLayouts`; afterward the reader stores its old false answer.
Every later read can return that answer despite the completed invalidation.
`sync.Map` prevents map corruption but does not serialize this sequence.
This is a separate defect from missing invalidation at lifecycle boundaries.

### [MEDIUM] Revert and rebase changes are unpinned

**Location:** internal/fabricengine/mergestateactive_integration_test.go:97

**Issue:** The Goal restores the two-repository disposition for cherry-pick, revert and rebase.
The added probe row and status-seam row exercise cherry-pick alone.
No two-repository regression asserts the result for the other operation markers listed in `gitOperationMarkers`.
Those cases belong in the existing probe scenarios, alongside the preserved merge and conflicted-index cases.

## Rejected

- The tests lens's no-git-on-repeat finding does not establish a useful failing assertion as proposed.
  The layout reader reaches `showFile`, which reads git objects in process, and local gitrepo reads use go-git.
  Counting subprocess starts alone would not distinguish the cache from repeated uncached reads.
  The rubric exempts pure-cache mutation survivors; observable invalidation defects and their checks are covered above.

## Review evidence

The board entry matches the decision record's Goal, and no amendments or declared PATTERN changes were supplied.
The changed one-repository operation-classification branch is unchanged in behavior.
The two-repository branch retains the conflicted-index check and error propagation.
A linked worktree's `.git` file passes the new existence check.
Other filesystem errors proceed to the reader and retain warning behavior without caching the failure.
Fresh missing paths are not cached.
The new test steps introduce no sleeps or elapsed-time assertions.

The targeted gate invocation could not acquire a slot because its holder record was on a read-only filesystem.
No tests ran in this review; the supplied gate reports were considered advisory evidence.
The command requested tiers 0 and 1 for fabricengine and loomcli, filtered to the changed regression tests.
The failure was opening the gate holder record under the hub's `_bolt/.lym/gate` directory.
