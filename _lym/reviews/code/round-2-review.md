---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/shedarbiter/call.go:359
    summary: the refusal for a failed spawn-time Changes read is reached by no test, so PATTERN-refusal-way-forward's reaching test is missing
    fix: In internal/shedarbiter/call.go:359-362 give the Changes-read refusal its own message, such as "the worktree's uncommitted paths could not be read before the spawn". In internal/shedarbiter/call_test.go TestCall_Edits add a spawnChangesErr field, pass it as fakeWorktree.changesErr when the fixture is built, and add the row "uncommitted paths that cannot be read at spawn refuse and spawn nothing" with spawnChangesErr set and wantRefusal naming that message; the existing refusal branch already asserts the way forward and that nothing started.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shedarbiter/edits.go:846
    summary: an untracked file present at spawn that the arbiter edits and leaves uncommitted is not caught, since Changes compares only the path set and Dirty hashes only tracked paths
    fix: Record a content hash for each uncommitted path at spawn, untracked ones included, and compare hashes after the arbiter. For example, change loomengine.ArbiterAttempt.Changes (internal/loomengine/arbiter.go:313) to a map from path to SHA-256 of the file (or "deleted"), filled in settleFresh from Worktree.Changes by hashing each path under the worktree; in readEdits compare that map with one built the same way after the arbiter, in place of samePaths. Add a TestCall_Edits row where spawnChanges and afterChanges list the same untracked path with different content and the resume hands on with cause precondition and reason "uncommitted changes". Update the readEdits doc and the WorktreeSeam.Changes and ArbiterAttempt.Changes docs to say the content is compared, and the summary's What changed bullet on the spawn snapshot to name the untracked paths.
  - id: F3
    severity: MEDIUM
    class: decision
    location: internal/battencli/drive.go:526
    summary: Batten's arbiter still gets no worktree seam and stays verdict-only, though the board entry widens the arbiter for every recipe, and the record still holds no Added after Discussion entry licensing the exclusion
    fix: Leave the code in place and defer this finding to the run's parent. The parent either records the batten exclusion with lym loom decision add, or rules that batten's arbiter must be wired with a worktree seam too. Recording or refusing that call is the parent's and lies outside the fixer's write surface.
---

### [BLOCKING] The spawn-time Changes refusal has no reaching test

**Location:** internal/shedarbiter/call.go:359

**Issue:**
`settleFresh` now refuses at two sites before the spawn.
The `Worktree.Snapshot` failure is reached by the row "a worktree that cannot be read at spawn refuses and spawns nothing".
The `Worktree.Changes` failure at lines 359-362 is reached by no row:
`fakeWorktree.changesErr` is only ever set inside `onWait`, from `afterChangesErr`, so the spawn read always succeeds in the test.
`PATTERN-refusal-way-forward` requires every new refusal in a `internal/shed*` package to land with a test that reaches it and asserts its message.
The two sites share a message, so the existing row's assertion cannot tell which read failed;
a regression that drops the `err` check after `Changes()` and spawns over an unread path list would pass the suite.
This also answers the round's second focus question for the spawn side: no row reaches the spawn-time `Changes` read separately.

### [MEDIUM] An edited untracked file found at spawn passes as unchanged

**Location:** internal/shedarbiter/edits.go:846

**Issue:**
`readEdits` decides "uncommitted changes" from two inputs:
`maps.Equal(current.Dirty, spawned.Dirty)`, whose map `worktreeSnapshot` in `internal/loomcli/drivewiring.go` builds from `fab.DirtyPaths()`, which lists tracked paths only;
and `samePaths(changes, attempt.Changes)`, which compares the set of paths from `CodeWorktreeChanges` and never their content.
An untracked file present at spawn, such as a new file a failed row left behind, is in both path lists and in neither `Dirty` map.
When the arbiter edits it and leaves it uncommitted, both comparisons are equal, so a resume goes on with an uncommitted change the commit list does not show and the parent is never told about.
The fix fragment tells the arbiter to leave no uncommitted change behind, a new file included.
The `readEdits` doc says an uncommitted change is "either an edit to a tracked file or an untracked file".
The summary says Go enforces a clean commit.
None of them holds for this case.
A new untracked file the arbiter leaves is caught, by the path-set compare, and a new file it commits shows as a commit, so the round's first focus question is settled for those two cases.

### [MEDIUM] Batten's arbiter is still not widened, and the parent has not ruled

**Location:** internal/battencli/drive.go:526

**Issue:**
The board entry's Idea reads "Widen the arbiter in general, for every recipe".
`loadedArbiterDeps` in `internal/battencli/drive.go` builds `shedarbiter.Deps` with no `Worktree`, so batten's arbiter gets the no-fix fragment and edits nothing.
The summary's reason, that batten's worktree is the prime and has other writers, is the writer's own scope call.
Round 1 raised this as its F2 and deferred it to the parent.
`_lym/discussion/decision-record.md` still has no `Added after Discussion` entry, so the call is neither recorded nor refused, and the finding stands until the parent rules.

## Focus checks settled with no finding

- The fix-mode text forbids every mutating git command except add and commit.
  `loom-fragment-arbiter-fix.md` says the only git commands that change anything are `git add` and `git commit` on the branch found, and forbids every other command that changes the repository, its index, its refs or its worktree.
  The template's "What you never do" keeps push, amend, rebase, reset and branch switches, and the no-fix fragment keeps "no git command that changes anything".
  Not a defect.
- `TestCall_Edits` reaches each read after the arbiter separately: the snapshot (`afterSnapshotErr`), the uncommitted paths (`afterChangesErr`), `IsAncestor` (`ancestorErr`) and `CommitLog` (`logErr`).
  The two commit-read rows assert the reason "HEAD moved from aaa to bbb and the arbiter's commits could not be read", and the `CommitLog` row also asserts the question names the move.
  The `IsAncestor` row does not assert the question, but both failures build the same problem string, which `handOnWith` passes through `appendTo`.
  Not a defect.
- `lym gate test ./internal/shedarbiter` passed at HEAD.
