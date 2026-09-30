# Card 2 — fabric-add-drops-run-records

**Edit:**
- `internal/fabricengine#createWeftWorktree`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine/doc.go`

**Create:**
- `internal/fabricengine/forkrecords.go`
- `internal/fabricengine/add_runrecords_integration_test.go`

**Uses:**
- `plan:internal/shedrun#RunsRootRel`
- `internal/fabricengine#PathTracked`
- `internal/fabricengine#CommitWeftPaths`
- `internal/fabricengine#OriginRecordRel`

**Intent:**
A freshly forked weft branch drops its parent's shed run records (the whole `shedrun.RunsRootRel()` tree under the pair's anchor) in the pair's first weft commit, so a new pair never inherits another run's seed or status (Decisions 1, 4, 5).

Mechanism — the no-checkout fork, which deletes nothing on disk and so stays outside the Fabric Destruction Chokepoint:

1. `createWeftWorktree` forks with `worktree add --no-checkout -b <branch> <path> <startPoint>` (still through `containedWorktreeAdd`), and records `KindWorktreeCreated`/`KindBranchCreated` right after the fork succeeds, exactly as today, so any later failure reaches `Add`'s existing rollback with both recorded.
2. It then calls a new unexported helper in `forkrecords.go` with the weft worktree path and the scoped root (`ScopedPathspec(l.AnchorRel, []string{shedrun.RunsRootRel()})`).
   The helper populates the index from `HEAD` (`git read-tree HEAD`), asks `PathTracked` whether the fork point tracks anything under the root, and writes the working tree out:
   when nothing is tracked there, `git checkout-index -a` writes every entry;
   when something is, it removes the root from the index only (`git rm -r --cached -q -- <root>`), writes every remaining entry (`git checkout-index -a`), then resets the index to `HEAD` again (`git read-tree HEAD`), leaving the root's entries tracked but absent on disk.
   It reports whether the root was tracked.
   No file under the root is ever written to the new worktree's disk, which is the "no run records on disk" constraint: `shedrun` reads seeds from disk, so an index-only removal after a full checkout would not fix the bug.
3. `createWeftWorktree` returns that answer to `Add`.
   On the adopt path (`weftBranchAlreadyExists`) `Add` drops nothing and the answer is false (Decision 2).
4. At step 10c, `Add` passes `shedrun.RunsRootRel()` alongside `OriginRecordRel()` to `CommitWeftPaths` only when the root was tracked, so `git add` stages the root's deletions in the same commit, under the unchanged message `fabric: record parent branch for <slug>`.
   Passing the root when nothing is tracked there would be a hard `pathspec did not match` git error, which a fresh hub with no run records must never hit.
   The commit stays a positive-only file list through `CommitWeftPaths`, per the Fabric Git Invariant.

A failure in the helper, or anywhere after it, runs `Add`'s existing `rollbackAdd` unchanged, which removes the created weft worktree and branch; the parent's weft branch is never touched (Decision 6).
No new mutation kind is recorded: the drop has no observable effect outside the pair's own first commit, which `CommitWeftPaths` already records as `KindCommitCreated`.
Under `SkipGit` the commit is skipped as today, leaving the root's deletions unstaged in the new weft worktree alongside the uncommitted origin record.

Update `Topology.Add`'s own doc comment and step comments (the fork at step 8, the commit at 10c), `createWeftWorktree`'s doc comment, and the `Add` paragraph of `doc.go` with the drop step: why it happens, why the fork is no-checkout, and why the adopt path drops nothing.

`add_runrecords_integration_test.go` (`//go:build integration`, package `fabricengine_test`, on `hubforge` fixtures) covers every `fabricengine` acceptance criterion:
- a parent weft branch tracking `_lyx/shed/<a>/` and `_lyx/shed/<b>/` gives a pair whose weft branch is exactly one commit ahead of the fork point, tracks nothing under the root, has no such directory on disk, carries the origin record, and is pushed, while the parent weft branch still tracks both;
- a parent weft branch tracking nothing under the root gives the same single origin-record commit as today;
- adopting an existing weft branch that tracks run records leaves them tracked and on disk (reuse the adopt setup `TestAddRollback_AdoptedWeftBranchSurvives` uses);
- adding from a task pair (non-`main` parent) whose own weft branch tracks `_lyx/shed/<parent>/` drops it from the child;
- a failure injected after the drop (a portal blocker file, the injection `add_rollback_adopt_test.go` uses) rolls the pair back fully — no weft worktree or branch for the slug — and leaves the parent weft branch's tip unchanged.
Build the tracked run-record paths through `shedrun` constructors, never a hand-spelled `shed` segment.

**ImpactSummary:** Every `Add` caller (`lyx fabric add`, batten's `Worktree-Create`, `hubforge` fixtures) now gets a pair without the parent's run records; the adopt path, rollback and the origin commit's message are unchanged.
