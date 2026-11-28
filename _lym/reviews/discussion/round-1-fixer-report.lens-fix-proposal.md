```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens. I challenged the reviewer proposals the fixer carried out as proposed.

**Findings**

1. **MEDIUM**: D6 (from F1/F2), "For each other member of the prefix, with that member's own seams (D2), it then runs the checks `landUnderLock` runs for its own run: `ownerChecks`' reconcile verdict and owner read, `landCheck`, `handedOverStop`'s owner read, and landingshed's pull-request resolver."
   - **Summary:** these checks are actions, not reads. Run from the leader, they hand over, reconcile and change pull requests for another live run while that run waits for the lock.
   - **Evidence:**
     - `ownerChecks` calls `fz.handOver` when another machine owns the run (`internal/landingshed/finalize_owner.go:70-75`).
     - `handedOverStop` also ends in `handOver` (`finalize_owner.go:133`).
     - `reportVerdict` takes `fz.deps.RemoveFollowUp` (`finalize_owner.go:57`).
     - The run-scope reconcile "can fast-forward the task branch" (`finalize.go:471`).
     - `pullRequestLanding` "closes a recorded pull request that GitHub reports open, removes the record" and makes GitHub calls (`finalize_pr.go:243`, `:299`, `:307`).
   - **Breakage:** the proposal's own premise, "the member's own Finalize meets the same check under the lock and stops as today", fails. The leader has already handed that run over, removed its record or moved its task branch. The record should say the leader judges each member through read-only verdicts and does nothing on a member's behalf. Hand-over, record removal and pull-request actions stay with that member's own Finalize.

2. **MEDIUM**: D5 (from F10), "Before it first detaches, the leader writes a scratch marker naming the chain commit" and "restores a task worktree found clean and detached at the marker's commit".
   - **Summary:** bisection checks out several prefix commits. A leader that dies on any prefix other than the first is detached at a commit the marker does not name.
   - **Evidence:** D7 says "each step gates the middle prefix", which is a different chain commit at each step. D6's rebuilds after a moved head or a left-out member build new chain commits. The restore covers only "the marker's commit".
   - **Breakage:** that case falls outside the restore and outside the "no marker" Stuck, so the way forward is undefined. The fix should rewrite the marker at every detach, or have the marker name the chain, so the restore covers any commit the batch checked out.

3. **MEDIUM**: D7 (from F16/F5), "A leader that is its own batch's culprit ... writes its landing failure record from the prefix-1 gate's log and goes to its fixer on that log".
   - **Summary:** the prefix-1 gate ran on chain commit 1, which is the current parent tip merged with the leader's task head. The fixer works on the task branch, which holds only D1's lock-free merge of an earlier parent tip.
   - **Evidence:**
     - `runFixer` reads the merge record's `MergedTip` (`finalize.go:1217`, `:1228`).
     - The merge record was written by D1's lock-free `mergeParent` (`finalize.go:1432-1442`).
     - D6 rebuilds the chain "on the new head" when the parent moved.
   - **Breakage:** when the parent moved after D1's merge, the fixer gets a failure log from a tree its worktree does not hold, with a merge record naming a different tip. It may fail to reproduce the failure and spend attempts. Today's path, the D8 `failed` bullet, merges first and then gates. The proposal should either merge the current parent first or note that the log's tree can differ from the worktree's.

4. **LOW**: D2 (from F8), "A run that joins again at Finalize rewrites its entry with its new task head, and keeps the outcome only while its task branch tip is still the entry's task head."
   - **Summary:** a `failed` run's fixer commits move its task tip. If the passes after the fixer count as joining again, the `failed` outcome is dropped and a leader can batch the run.
   - **Evidence:**
     - D8: `failed` "keeps the outcome, so no leader takes it ... and lands alone".
     - D7's new criterion: "the next leader's batch does not take its entry".
     - `freeLock` frees the lock for the fixer, after which "the run then runs its passes again" (`finalize.go:676-689`).
   - **Breakage:** the record does not say whether those passes are a rejoin, so F8's rule and D8's land-alone rule disagree.

**Tool calls:** none beyond my context.
