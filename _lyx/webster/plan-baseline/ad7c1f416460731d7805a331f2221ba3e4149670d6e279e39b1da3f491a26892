# Card 11 — webster-reset-plan

**Create:**
- `internal/websterengine#ResetTarget`
- `internal/websterengine#ResetDeps`
- `internal/websterengine#ResetPlan`
- `internal/websterengine#PlanReset`
- `internal/websterengine#wayForwardSteps`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`

**Uses:**
- `internal/websterengine#RunWrites`
- `internal/websterengine#loadRunWrites`
- `internal/websterengine#State`
- `internal/websterengine#runEvidenceBases`
- `internal/websterengine#missingCommitsClause`
- `internal/websterengine#refuseMidMerge`
- `internal/websterengine#RunActive`
- `internal/websterengine#trackedRel`
- `internal/websterengine#isAncestor`
- `internal/websterengine#headSHA`
- `internal/websterengine#ParentBranchFunc`

**Intent:**
D6's Webster half, in a new `internal/websterengine/reset.go`: decide, read-only, what `lyx webster reset` may do, and refuse with a way forward before fabric is ever called.
Webster runs no mutating git (PATTERN-fabric-git); every probe here is a read through `gitexec.Run` or `gitrepo`.

`ResetTarget` has two constants, `ResetToStart` ("start") and `ResetToPreFix` ("pre-fix"); no raw SHA is accepted (A6).
`ResetDeps` carries the told `Geometry`, the loaded `*State`, the `shuttleengine.Engine`, `ParentBranch ParentBranchFunc`, and `Branch func() (string, error)`, the checked-out branch as fabric reads it, erroring on a detached HEAD.
`ResetPlan` carries the target, its SHA and `OwnPaths`, the worktree-relative tracked paths the run wrote.

`wayForwardSteps` renders `way forward: 1) <a>; 2) <b>` for two or more steps and `way forward: <a>` for one; it is the one renderer of an ordered step list, so each step is a distinct action and none restates another (D8).

`PlanReset` refuses, each with its own way forward, in this order:
1. run lock held (`RunActive`): transient; "wait for it to finish, or check `lyx webster status`";
2. no state (nil `State`): "run `lyx webster run` first";
3. a merge in progress (`refuseMidMerge`'s own text);
4. detached HEAD, a checked-out branch equal to the parent branch, or a branch the caller reports is not the pair's own: each names the task worktree's own branch to check out with `git switch`, then re-running the verb;
5. no recorded target: for `start`, no batch recorded a start commit, way forward `lyx webster run --fresh`; for `pre-fix`, `State.PreFixHead` is empty, way forward `lyx webster run`;
6. a recorded commit missing from the repository: `missingCommitsClause`;
7. a target that is not an ancestor of HEAD: the branch was rewritten under the run; way forward `lyx webster reset --to start` for `pre-fix`, `lyx webster run --fresh` for `start`;
8. a dirty tracked path outside the run's own paths: names each path; way forward: commit it on the task branch, or restore it with `git checkout -- <path>`, then re-run the verb.

Target resolution:
`start` is `runEvidenceBases`' `Start`;
when starts are recorded but none is the oldest of all, it is the octopus merge-base of the recorded starts (`git merge-base --octopus`, read-only), the case `--fresh` names today;
`pre-fix` is `State.PreFixHead`.
Own paths: `loadRunWrites` events whose `Succeeded` is true, Master's and every fork's, mapped through `trackedRel` to tracked worktree-relative paths; a failed write is not evidence (R4 is by path, not content).

Bound: the plan can name only a run-recorded commit that is an ancestor of HEAD on the task's own branch, and exempts from the dirt check only paths the run's own transcripts record a successful write to; it never reads `--force`.

Tests:
`reset_test.go` (untagged) pins `wayForwardSteps` for one, two and three steps and the run-lock, no-state and no-recorded-target refusals;
`reset_integration_test.go` (scratch repositories under `t.TempDir()`, `TestMain` already hermetic) covers the merge, detached, parent-branch, missing-commit, not-ancestor and foreign-dirt refusals, the octopus merge-base when two recorded starts diverge, and a plan whose `OwnPaths` holds a succeeded write and omits a failed one.

**ImpactSummary:** new read-only planning code; no existing function changes.
