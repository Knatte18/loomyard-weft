---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md#Constraints (Fabric Destruction Chokepoint bullet)
    summary: The index-only removal the constraint offers as the escape from the destruction chokepoint leaves the parent's run records on disk, because the create path's worktree add -b has already checked them out; that fails Acceptance criterion 1 and leaves the bug in place.
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#Decisions (Decision 2)
    summary: The rationale that an adopted weft branch is always the pair's own does not hold for a weft branch left behind by a crash between the fork and the origin commit, which is an unmodified fork of the parent and carries the parent's run records into the adopted pair.
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md#Decisions (Decision 5) and #Acceptance criteria (bullet 1)
    summary: Decision 5 leaves the commit message as either of two options, and Acceptance criterion 1 never asserts the single-commit rule on the drop path, so Plan has nothing to test the rule against.
  - id: F4
    severity: NIT
    location: _lyx/discussion/decision-record.md#Decisions (Decision 4)
    summary: The sentence listing shedrun's imports to argue there is no cycle is a Plan-time dependency fact the writer stencil says not to gather; support-log Interview turn 5 already records it.
---

### [BLOCKING] Index-only removal does not keep the records off disk

**Location:** `_lyx/discussion/decision-record.md`, `## Constraints`, the Fabric Destruction Chokepoint bullet; also Decision 1 ("immediately after that fork") and support-log Question ledger #6.

**Issue:** `Add`'s create path runs `createWeftWorktree`, which is `git worktree add -b <weft> <path> <parentWeft>` (`internal/fabricengine/weftwiring.go`).
That command checks out the parent's whole tree, `_lyx/shed/` included, before any drop can run.
An index-only removal after that point (`git rm --cached`) untracks the files but leaves them on disk.
`shedrun` reads seeds with `os.ReadFile` (`internal/shedrun/seed.go`), and `List` and `runSegment` stat the on-disk directory, so the child's loom run still finds the foreign seed and the reported failure persists.
Acceptance criterion 1 ("has no such files on disk") would also fail.
The constraint says the index-only route "avoids the question", which is true of destruction but false of the fix, so the record points Plan at a mechanism that contradicts its own first criterion.
Decision 1's "immediately after that fork" frames the drop as a step after the checkout, which reinforces the same reading.

An index-only removal does work if the removal happens before the tree reaches disk.
Verified in a scratch repo: `git worktree add --no-checkout -b c <path> <start>`, then `git reset`, `git rm -r --cached _lyx/shed`, `git checkout-index -a` writes a worktree with no `_lyx/shed/`, deletes nothing, and a later `git add -- <origin>` plus `git commit -- <origin> _lyx/shed` produces one commit that drops the subtree and adds the origin record.

**Fix:** State the requirement plainly: the parent's run records must never land on the new weft worktree's disk untracked.
Name the two mechanisms that satisfy it: a fork that keeps the subtree off disk (deletes nothing, so outside the chokepoint), or a working-tree delete routed through `destroy.go` with its checks.
Say that an index-only removal after a normal checkout satisfies neither.
Reword Decision 1 so the drop is part of creating the new branch, not a step after it, and update Question ledger #6 to match.

### [MEDIUM] The adopt path can adopt an unmodified fork of the parent

**Location:** `_lyx/discussion/decision-record.md`, `## Decisions`, Decision 2.

**Issue:** `Add` takes the adopt path when the local weft branch exists (`weftBranchExists`) and the warp branch does not.
`Remove` deletes the weft branch (`removeWeftWorktree` with `alsoDeleteBranch` true), and `rollbackAdd` deletes a weft branch this `Add` created, so neither normally leaves one behind.
A crash between the fork (step 8) and the origin commit (step 10c) does: no rollback runs, and the leftover weft branch is the parent's weft tip with its run records.
A batten retry of `Worktree-Create` stops at `ErrBranchExists` on the warp branch, so batten never adopts it on its own.
An operator who follows that error's remedy (`git branch -D` on the warp branch), removes the leftover directories, and re-adds, lands on the adopt path.
The adopted pair then carries the parent's records, and when the parent is prime it carries batten's seed for this same slug, which is the reported failure.
The rationale "an existing branch is the pair's own" states this as always true.

**Fix:** Narrow the rationale to what holds: a weft branch that survives `Remove` and `rollbackAdd` is the pair's own.
Add an Open risk naming the crash window, the manual recovery that reaches the adopt path, and the remedy: delete the leftover weft branch too before re-adding.

### [LOW] The single-commit rule has no test anchor

**Location:** `_lyx/discussion/decision-record.md`, Decision 5 and Acceptance criterion 1.

**Issue:** Decision 5 says the drop lands in the origin-record commit, with message `fabric: record parent branch for <slug>` "or a message naming both".
That leaves the message undecided.
Acceptance criterion 2 pins the no-drop case to "the same single origin-record commit as today", but criterion 1, the drop case, never says the drop and the origin record arrive in one commit, so a two-commit implementation passes every criterion.
The lock-wait verdicts in Decision 7 and the `battenshed` criteria agree with each other and need no change.

**Fix:** Pick one message: keep `fabric: record parent branch for <slug>` for both cases.
Add to criterion 1 that the new weft branch is exactly one commit ahead of the fork point.

### [NIT] Decision 4 carries a dependency-list fact

**Location:** `_lyx/discussion/decision-record.md`, Decision 4, last sentence.

**Issue:** "`shedrun` imports only `lyxcwd` and `lyxdirs`, so the new import creates no cycle" is a dependency-list fact, which the writer stencil tells Discussion not to gather and Plan computes fresh.
It carries no requirement: the requirement is that `Add` calls the `shedrun` constructor, which the Decision's first two sentences already state.
Support-log Interview turn 5 already records the cycle check, so dropping it loses nothing.

**Fix:** Delete the sentence from `decision-record.md`.

## Focus verdicts

- Focus 1 (chokepoint vs. "no such files on disk"): a defect, recorded as F1.
- Focus 2 (adopt path): a defect, recorded as F2.
- Focus 3 (single-commit rule, lock-wait verdicts): the lock-wait verdicts (Stuck at the bound with a reason naming the path and the wait, the existing cancel error, a hard error on `Acquire` failure, release on every exit) are consistent between Decision 7 and the `battenshed` criteria.
  The single-commit rule is under-tested, recorded as F3.
- Focus 4 (leanness): Decision 4's import sentence is relocatable, recorded as F4.
  Decision 7's sleep seam is a testability requirement the unit criteria depend on ("contended for N polls"), and its info log line is an operator-visible behaviour; both stay.
  The Auto-mode assumption that the hub already cleared the blocked pairs by hand is the premise that makes the Scope's "inherited copies are inert" safe, so it carries a constraint and stays.
