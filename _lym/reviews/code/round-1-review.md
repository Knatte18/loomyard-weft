---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shedarbiter/edits.go:688
    summary: readEdits never sees an untracked file, so an arbiter that leaves a new file uncommitted resumes the run on a fix the task branch does not hold
    fix: Make the worktree check see untracked paths in the task's code checkout. Extend WorktreeSeam with a read of the code checkout's untracked paths, wired in internal/loomcli/arbiter.go's arbiterWorktreeSeam through fabric's read-only CodeWorktreeChanges (internal/fabricengine/diff.go:120), and record that set in the spawn snapshot. In readEdits, set the problem "the arbiter left uncommitted changes in the worktree" when the untracked set differs from the one recorded at spawn. Add a TestCall_Edits row with a new untracked file, expecting a precondition hand-on. Correct the WorktreeSeam.Snapshot doc comment and the summary's "hash of each tracked uncommitted path" to match.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/battencli/drive.go:526
    summary: The board entry widens the arbiter for every recipe, but batten's arbiter gets no worktree seam and stays verdict-only, which no Added after Discussion entry licenses
    fix: Leave the code in place and defer this finding to the run's parent. The parent either records the batten exclusion with lym loom decision add, or rules that batten's arbiter must be wired too. Recording or refusing a call is the parent's job and outside the fixer's write surface.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedarbiter/call_test.go:412
    summary: No test reaches readEdits's three read-failure paths after the arbiter (the Snapshot error, which includes a detached HEAD, the IsAncestor error and the CommitLog error)
    fix: Give fakeWorktree separate errors for the spawn read, the post-arbiter Snapshot read, IsAncestor and CommitLog. Add TestCall_Edits rows for each post-arbiter failure, each expecting a precondition hand-on whose reason contains the matching text (could not be read after the arbiter, or the arbiter's commits could not be read).
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedarbiter/edits.go:675
    summary: When HEAD moved but IsAncestor or CommitLog fails, the hand-on lists no commits and does not say HEAD moved, so the parent gets no sign that arbiter commits exist to keep or revert
    fix: In readEdits, have both commits-could-not-be-read problem texts also name the move, so the text reads HEAD moved from the spawn SHA to the current SHA, then the read error, so appendTo's note and the reason point the parent at the range. Cover it with the rows F3 adds.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedarbiter/call.go:327
    summary: No row covers a resume over uncommitted changes after commits, whose hand-on must still list the commits, or an agent's own hand-on whose question gains the Check the worktree first note
    fix: Add two TestCall_Edits rows. The first is a resume whose after-snapshot has a moved HEAD and a dirty path, with descends true and commits set; it expects a precondition hand-on with wantCommits equal to the commits and wantQuestion containing the commit. The second is handOnVerdict over a dirty after-snapshot; it expects wantQuestion containing Check the worktree first and uncommitted changes.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedarbiter/seam.go:40
    summary: No test fails if Seam's Judge drops envelope.Commits, which would silently drop the commits from the driver's settle record and stop report
    fix: In internal/shedarbiter, add an assertion that Seam(deps, runID, ...).Judge returns an ArbiterOutcome whose Commits equal the settled envelope's commits. Fold it into an existing call-fixture test, for example by driving the TestCall_Edits row "a resume after commits with a goto lists them" through Seam rather than NewCall, and do not add a new test function.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/arbiter.go:166
    summary: The lym loom arbiter output's new commits key, and its rendering as an empty list when the envelope has none, have no test
    fix: Add a table test of arbiterEnvelopeFields in internal/loomcli, or a row in an existing verb-output test there. Its rows are an envelope with Commits set, expecting commits equal to that list, and one with nil Commits, expecting an empty non-nil []string.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shedarbiter/prompt_test.go:32
    summary: The redundancy report lists TestPrompt as contained by TestCall_Edits, but TestPrompt pins what TestCall_Edits never asserts, and it carries no keep marker
    fix: Add a line directly above func TestPrompt reading //testtiming:keep pins every marker value, absent inputs reading none, the parent and edit directives, and the worktree and Arbiter-Attempt trailer filled into the fix fragment, which TestCall_Edits does not assert
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: contracts/stencils/loom/loom-template-arbiter.md:86
    summary: In fix mode the general rule never to run a mutating git command is replaced by a list (push, amend, rebase, reset, branch switch), so pull, merge, cherry-pick, stash and clean are no longer named as forbidden
    fix: In loom-fragment-arbiter-fix.md, add a bullet stating that the only git commands that change anything you may run are git add and git commit on the branch you found, and that you never run any other git command that changes the repository, its index, its refs or its worktree. Keep the template's Push, amend, rebase, reset or switch a branch line.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/doc.go:441
    summary: The loomcli package doc's Arbiter section does not say that loom wires the worktree seam so its arbiter may commit one fix, that attempt.json records the worktree snapshot, or that the envelope and the verb's output list commits
    fix: In the # Arbiter section of internal/loomcli/doc.go, add sentences saying that the verb wires Deps.Worktree through fabric's read-only reads, so the arbiter of a loom-engine recipe may commit one fix on the task branch; that attempt.json records the worktree snapshot taken at spawn; and that envelope.json and the verb's output list the arbiter's commits.
---

The diff answers the board entry's three questions as follows.
The arbiter may edit the task's code, tests and docs, while plan, record and PATTERN changes stay with the repair verbs.
It commits on the branch it found, and a resume after its commits needs a goto.
Its commits show in the envelope, the verb's output, the re-step settle record, the stop report's attempt line and the hand-on question.
The worktree seam goes only through fabric's read-only reads, and lym code runs no mutating git, so PATTERN-fabric-boundary holds.
The directive declares no PATTERN change, and the commit range carries no trailer.

### [BLOCKING] Untracked files escape the worktree check

**Location:** internal/shedarbiter/edits.go:688

**Issue:** `readEdits` compares `Dirty` maps, which `worktreeSnapshot` (`internal/loomcli/drivewiring.go:396`) builds from `fab.DirtyPaths`.
That is `git status --porcelain -z --untracked-files=no` (`internal/fabricengine/dirtiness.go:67`), so new files never appear.
An arbiter that adds a file and commits with `git commit -a`, or leaves the file uncommitted, passes the check.
The run's gates then build and test code the task branch does not hold.
This breaks the entry's second question, that an edit is committed and then checked by the run's own gates.
It also contradicts the fix fragment's "An uncommitted change ... hands the stop on", the `doc.go` rule that every change is left committed, and the summary's claim that Go enforces a clean commit.

### [MEDIUM] Batten's arbiter is not widened

**Location:** internal/battencli/drive.go:526

**Issue:** The entry's Idea reads "Widen the arbiter in general, for every recipe".
`loadedArbiterDeps` sets no `Worktree`, so batten's arbiter stays verdict-only.
The summary gives a sound-looking reason: batten's worktree is the prime, which has other writers.
But this is the writer's own scope choice, and the record has no `Added after Discussion` entry for it.
Only the parent can accept or refuse the choice.

### [MEDIUM] readEdits's read-failure paths are untested

**Location:** internal/shedarbiter/call_test.go:412

**Issue:** `fakeWorktree` never fails `IsAncestor` or `CommitLog`.
Its `snapshotErr` also fails the spawn read, so no row reaches the read after the arbiter.
These three paths decide a precondition hand-on at a trust boundary.
One of them is the detached-HEAD case the focus names, because `gitrepo.CurrentBranch` errors on a detached HEAD.

### [LOW] A failed commit read hides that commits exist

**Location:** internal/shedarbiter/edits.go:675

**Issue:** When HEAD moved but `IsAncestor` or `CommitLog` fails, the result has no commits.
Its problem text says only that the commits could not be read, so the parent is not told that the branch moved.

### [LOW] Combined edit outcomes are uncovered

**Location:** internal/shedarbiter/call.go:327

**Issue:** No test covers a resume over uncommitted changes after commits, which goes through `handOnWith(attemptEdits{commits: ...})`.
No test covers an agent's own hand-on over a worktree problem, which goes through `appendTo`'s "Check the worktree first" note.

### [LOW] Seam's commits passthrough is uncovered

**Location:** internal/shedarbiter/seam.go:40

**Issue:** `TestCall_Edits` stops at the envelope, and `TestGoDriver_Settle` builds `ArbiterOutcome` by hand.
Nothing pins the mapping in between.

### [LOW] The verb's commits key is uncovered

**Location:** internal/loomcli/arbiter.go:166

**Issue:** No loomcli test references `arbiterEnvelopeFields`.
The new key and its empty-list rendering are untested.

### [LOW] TestPrompt needs a keep marker

**Location:** internal/shedarbiter/prompt_test.go:32

**Issue:** The report lists `TestPrompt` as contained by `TestCall_Edits`.
`TestCall_Edits` checks only the fragment heading.
`TestPrompt` also asserts every marker value, the absent inputs reading `none`, the parent and edit directives, and the worktree and run ID filled into the fix fragment.
It pins what coverage cannot see, so it stays, with a marker.

### [LOW] The fix-mode stencil drops the general no-mutating-git rule

**Location:** contracts/stencils/loom/loom-template-arbiter.md:86

**Issue:** The old line "Run a git command that changes anything" now survives only in the no-fix fragment.
In fix mode only push, amend, rebase, reset and branch switches are named.
A `git pull` or `git merge` of the parent passes `readEdits`, since the result descends from the spawn HEAD.
Its commits would then be reported as the arbiter's own.
The entry asked for the arbiter to commit its fix, not for the wider git surface.

### [LOW] loomcli's package doc is stale

**Location:** internal/loomcli/doc.go:441

**Issue:** The `# Arbiter` section describes `attempt.json` and the verb's output, but not the worktree seam loom now wires, the snapshot, or `commits`.
The sibling doc files, the cobra help and `docs/overview.md` were all updated.

## Rejected

- code-tests NIT: the fragment-heading assertions are duplicated between `TestPrompt` and `TestCall_Edits`. Rejected, because each pins a different link. `TestPrompt` pins `Prompt`'s choice of fragment from `MayEdit`, and `TestCall_Edits` pins the call's setting of `MayEdit` from `Deps.Worktree`.
