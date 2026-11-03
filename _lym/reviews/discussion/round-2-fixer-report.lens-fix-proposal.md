```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as written (F1, F2, F3, F5, F6, F7 option (a), F8, F9 option 1, F10, F11, F13, F14, F15) and built the cases below.

**[MEDIUM] F7 as carried out, D1 step 3: the reused records worktree has no rollback guard**
- **Changed text:** "`Add` reuses it instead of refusing it: it adds no weft worktree, …, makes only the task worktree …".
- **Summary:** if `Add` fails after it reuses a records worktree it never made, its rollback deletes that worktree, and any uncommitted run records go with it.
- **Evidence:** every failure path in `internal/fabricengine/add.go` after the warp worktree exists calls `rollbackAdd`.
  - `rollbackAdd` step (1) calls `removeWeftWorktree(rec, l, slug, weftBranch, true, !weftBranchAdopted, false, …)` on `RecordsWorktreePath(l, slug)`, the reused path.
  - Its doc says it removes the worktrees and keeps only a pre-existing weft *branch*, not a pre-existing weft *worktree*.
  - Case: the reuse succeeds, then `WireJunctionsWith` or `writeLaunchers` fails, and the rollback removes the standing records worktree.
  - D1 does not say the rollback leaves a reused records worktree in place.

**[MEDIUM] F5 as carried out, D8: the warp probe's answer also feeds `Add`'s fetch**
- **Changed text:** "An adopted local warp branch's is read by the warp probe, which `Add` skips for such a branch except under the adopt-only option, which runs it."
- **Summary:** running `probeWarpLeftover` for an adopted local warp branch does more than answer D8's question, so it can make the re-adopt fail every pass.
- **Evidence:**
  - `probeWarpLeftover` (`internal/fabricengine/remoteleftover.go:162-176`) returns origin's tip whenever origin holds the branch and the weft is live, and `add.go` stores it in `warpAdoptTip`.
  - `add.go:413-418` then runs `git fetch --no-tags origin refs/heads/&lt;warp&gt;:refs/remotes/origin/&lt;warp&gt;`. That refspec has no `+`, so git refuses a non-fast-forward update of the tracking ref.
  - Case: a stale tracking ref from a former run, on a tip origin's current task branch does not descend from. The fetch fails, `Add` fails with a transient error, and every pass fails the same way. That is the Goal's forbidden outcome.
  - The record should say that under adopt-only the warp probe's answer serves only D8's drop and never sets `warpAdoptTip`.

**[LOW] F1 as carried out, D2's `ErrRemoteLeftover` way forward and D8: the unset can fail on a branch with no upstream**
- **Changed text:** "then `git -C &lt;records repo&gt; branch --unset-upstream &lt;weft branch&gt;`", and in D8, "unsets the branch's upstream config … in one step".
- **Summary:** `git branch --unset-upstream` exits fatal ("has no upstream information") on a branch with no upstream config, and both changed texts can meet one.
- **Evidence:**
  - D2's not-live restore makes the weft branch with `git branch &lt;weft branch&gt; &lt;tag&gt;`, which sets no upstream. If that branch later meets `ErrRemoteLeftover`, the named hand step ends in a fatal error.
  - D8 applies the unset to every forked branch, and a fresh fork has no upstream config, so an implementation that runs `--unset-upstream` there fails `Add`.
  - Neither text says a missing upstream is a no-op.

**[LOW] F3 as carried out, D7's new bullet: a skipped push now strands the commit locally**
- **Changed text:** "the synchronous `commit-records` push included, … exits without pushing when a removal holds it exclusive".
- **Summary:** a `commit-records` commit made after D4 read the local tip is no longer pushed. The removal then deletes the local weft branch, and no archive tag covers that commit, so it is lost. Before this change it landed on origin, where D6's lease kept it.
- **Evidence:**
  - D4 archives "the local tip" read after "the pending records are committed".
  - The PATTERN entry `PATTERN-push-both-sides` says the code branch goes along and a refused push leaves a failed-push record. Under the new rule a skipped push leaves no record either.
  - The record states no bound saying such a commit is given up on purpose. The cost is small, since it only applies to a pair being cancelled.

No case found against F2's skip list or new answers, F6's `--remove` form, F8, F9, F10, F11, F13, F14 or F15. For F6 I checked that `Remove` finishes a pair whose task worktree is gone and reaches `ErrPairNotFound` only when nothing of the pair is left.

Tool calls:
1. `internal/fabricengine/remove.go` lines 120-175 and its `ErrPairNotFound` uses: whether `cancel --remove` clears a pair with both worktrees absent (for F6). My context had only the review's line citations.
2. `probeWarpLeftover` in `internal/fabricengine/remoteleftover.go`: what running the warp probe on an adopted warp branch returns (for F5). My context did not hold that function.
