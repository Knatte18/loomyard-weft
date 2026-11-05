---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/originreconcile.go:666
    summary: The diff declares the detachedBranch constant for the detached-HEAD marker but leaves a repeated literal of it in the same package, against PATTERN-named-constants.
    fix: In internal/fabricengine/originreconcile.go line 666 replace the literal "(detached)" in the condition `entry.Branch == "(detached)"` with the detachedBranch constant declared in internal/fabricengine/worktreelist.go. Leave the literal in matchparent_test.go alone, since a test fixture spells the wire value on purpose.
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/fabricengine/pairbranch.go:57
    summary: PairTaskBranch reads a worktree's branch only when it is the current prefix's name or the bare slug, so a branch_prefix changed from one non-empty value to another leaves live pairs under the first prefix unrecognised, which the record's "can be set at any time" does not carve out.
    fix: Leave the code in place and defer this finding to the run's parent, who records the narrowing with `lym loom decision add` or refuses it. If the parent refuses it, the change is to have pairTaskBranchLocal accept the task worktree's checked-out branch when it is a branch of the pair, for example one whose records branch exists in the records repository, instead of only branches in pairBranchNames.
  - id: F3
    severity: LOW
    class: design
    location: internal/fabricengine/remove.go:208
    summary: A pair with only its launcher directory left locally is named by the current prefix without asking origin, so a bare-slug pair's branches stay on origin after the first Remove.
    fix: In internal/fabricengine/pairbranch.go have pairTaskBranchLocal's found result reach pairState as a new field (for example warpBranchFound) set in inspectPair. In refuseWhenNothingRemains ask originPairBranch when no worktree and no local branch of the pair remains, launchers and stray path not counted, while still returning ErrPairNotFound only when launchers are also gone and origin holds nothing. In removePair adopt onOrigin whenever warpBranchFound is false and onOrigin is non-empty, instead of only when localPartRemains is false. Add a row to TestBranchPrefixChange_PairWithNothingLeftLocally that leaves only the launcher directory over a bare-slug pair on origin and expects Remove with remote to delete both bare-slug branches there.
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/pairbranch.go:70
    summary: The skip of a records worktree whose repository root cannot be resolved has no test, so the half-torn-pair behavior fix F6 added is unpinned.
    fix: Add a row to TestPairTaskBranch_ReadsThePairItself in internal/fabricengine/pairbranch_integration_test.go that keeps the records worktree directory at RecordsWorktreePath present while RecordsRepoRoot fails for the hub, and expects PairTaskBranch to return the name a local branch carries instead of an error. Add an assertion in the same row that PairParts for the slug completes without error.
---

### [BLOCKING] A detached-HEAD literal survives next to the new constant

**Location:** `internal/fabricengine/originreconcile.go:666`
**Issue:** `PATTERN-named-constants` asks for a name that can go stale to be one named constant, declared once and referenced everywhere.
The diff adds `detachedBranch` in `internal/fabricengine/worktreelist.go` and uses it in `parseWorktreePorcelain` and `worktreeBranchAt`.
`originreconcile.go:666` still compares `entry.Branch` against the literal `"(detached)"`, the same value the parser writes into `WorktreeEntry.Branch`.
A change to the parser's marker would update the constant and silently break that comparison.

### [MEDIUM] A prefix changed between two non-empty values is not covered by the worktree read

**Location:** `internal/fabricengine/pairbranch.go:57`
**Issue:** The decision record's chosen fix says the pair's worktree is the source of its branch name, and that `branch_prefix` "can be set at any time".
`pairTaskBranchLocal` reads a worktree's branch only when `slices.Contains(names, branch)`, where `names` is the current prefix plus slug and the bare slug.
A pair made under `hanf/` stands on `hanf/<slug>`.
After the prefix changes to `other/` that branch is neither name, so the worktree step is skipped, the local-branch step finds nothing, and the pair is named `other/<slug>`.
Remove, discard and the pull request lookup then miss the real branch, which is the failure the record describes for the empty-to-set change.
The change description records this as a design decision, and the record's acceptance covers only the empty-prefix case, so the code is not shown to be wrong.
It is a recorded-nowhere narrowing of the stated rule, which the parent should settle.

### [LOW] A pair with only launchers left is named by the current prefix without asking origin

**Location:** `internal/fabricengine/remove.go:208`
**Issue:** `removePair` adopts the name origin holds only when `!pair.localPartRemains()`, and `localPartRemains` counts the launcher directory.
`refuseWhenNothingRemains` returns early for the same reason and never asks origin.
For a half-torn pair made under the empty prefix whose only local remnant is its launcher directory, with its bare-slug branches still on origin and `branch_prefix` now set, `inspectPair` names it `<prefix><slug>`.
`PairTaskBranchOrOrigin` would have found the bare slug, because it ignores launchers.
The remote teardown job then targets `<prefix><slug>` and `<prefix><slug>-weft`, which are absent, and the bare-slug branches stay on origin.
The first Remove reports success, and a second Remove finds nothing local and then names the pair from origin.
The case is narrow, and the next run clears it.

### [MEDIUM] The skip of an unresolvable records repository in PairTaskBranch has no test

**Location:** `internal/fabricengine/pairbranch.go:70`
**Issue:** Fix F6 added the `rootErr == nil` guard so that a records worktree directory whose repository root cannot be resolved leaves the records step to the local-branch step, and `inspectPair`, `RemoveRefusal`, `Remove`, `PairParts` and `DiscardTaskBranch` still finish for a half-torn pair.
No row in `TestPairTaskBranch_ReadsThePairItself` or any other test in `internal/fabricengine` makes `RecordsRepoRoot` fail while the records worktree path is present.
The rows only exercise a resolvable records repository.
This is behavior the diff adds with no coverage.
