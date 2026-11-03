---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/remove_leftoverdir_integration_test.go:166
    summary: Remove's refill pass has no test for a delete that fails, so the refilledKept to StrayPath path is unpinned
    fix: Add a test beside TestRemove_DeletesTaskWorktreeDirRefilledAfterItsRemoval that refills the directory in the teardown hook and makes the removal fail with a non-late-writer error through the leftoverSeam remove seam. Assert Remove returns nil error and res.StrayPath equals the task worktree path.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/prune_unowned_integration_test.go:129
    summary: Prune's third pass has only a positive test; the skips for a live registered pair, a hidden directory, a weft-suffixed directory and a reserved name are unpinned
    fix: In TestPrune_ReportsANonWorktreeDirectoryAtAPairPath, or a sibling table test, also create a registered pair (hubforge.AddPairWith), a directory named .hidden and a reserved hub name, and assert Prune returns no entry for them. A broken liveWarpSlugs skip would otherwise report every real pair as Unowned with the suite green.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remove.go:1001
    summary: PairWorktreeStands reads false for a live pair when git worktree list fails, so the rename and project-change refusal fails open where the old stat check could not
    fix: In PairWorktreeStands, call List(l.WorktreePath()) directly and, when it errors with the path present, return true so an unlistable repository keeps refusing; otherwise return listsLinkedWorktree(entries, target). Correct the doc comment to say so.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/wire.go:365
    summary: The ErrTaskPathTaken halt says the directory is not a worktree and to delete it, but Add raises that error for any existing path, including a registered worktree that failed ResolveWorktree
    fix: Before returning the precondition error in createRefusal's caller at wire.go:986, check fabricengine.PairWorktreeStands(location, slug); when it is true, return the original error instead. Add a TaskPathTaken row for that case in TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout, or word the Unmet text without claiming the directory is no worktree.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/boardcli/cli.go:148
    summary: No boardcli test shows a plain directory at a pair's path no longer refuses a rename or project change; reverting the seam to os.Stat would pass
    fix: Add a step to stepProjects in internal/boardcli/verbs_test.go that creates a plain directory at another slug's pair path and expects its project change or rename to succeed.
---

## Rejected

- The registered-worktree guard at `remove.go:330` has no test row (lens `code-tests`, part of its F-A): the exclusive teardown lock makes a re-registration at the path unreachable, so a row would test a state the code's own guarantees rule out.
- `code-affected-set` returned no findings: I found the affected set covered as well.
- The cancel's first Expected bullet allows a directory to remain after a late writer outpaces the second look: the writer's own summary names the residual window, and the warning in `cancel.go` covers it, so I raise nothing.

### [MEDIUM] Remove's refill pass has no test for a delete that fails

**Location:** internal/fabricengine/remove_leftoverdir_integration_test.go:166
**Issue:** `removePair` sets `refilledKept` and reports `StrayPath` when the second removal fails (`remove.go:331-334`, `358`). The only refill test succeeds at the delete, so this behavior can break with the suite green.

### [MEDIUM] Prune's third pass has no negative rows

**Location:** internal/fabricengine/prune_unowned_integration_test.go:129
**Issue:** The one test parks a single stray directory. Nothing asserts that a live pair, a hidden directory, a weft-suffixed directory or a reserved name stays unreported (`prune.go:186-191`), though a broken skip would flag every real pair.

### [LOW] PairWorktreeStands fails open

**Location:** internal/fabricengine/remove.go:1001
**Issue:** A failed worktree listing makes a live pair read as absent, so a rename or project change that should refuse goes through. The helper's conservative direction suits deletions, not this guard.

### [LOW] The path-taken halt can tell an operator to delete a registered worktree

**Location:** internal/battencli/wire.go:365
**Issue:** `Add` returns `ErrTaskPathTaken` for any existing path (`add.go:255`). A registered worktree that `taskWorktreePresent` read as absent, because `ResolveWorktree` failed, then gets the text "is not a worktree ... delete it". This is plausible but not shown to be reachable, so it stays LOW.

### [LOW] The leftover-directory side of the rename seam is untested

**Location:** internal/boardcli/cli.go:148
**Issue:** `TestCLI/Projects` now builds a real worktree, which a plain `os.Stat` would also accept. Only the `fabricengine` function is pinned for a plain directory, not the boardcli wiring.
