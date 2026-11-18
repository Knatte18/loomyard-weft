```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F24 to F29) by building cases where each proposed change is wrong.

1. **MEDIUM: D4's in-merge rule only covers `MERGE_HEAD`, but git also refuses a partial commit during a cherry-pick or a rebase.**
   - Changed text: D4's new bullet, "So while `MERGE_HEAD` is present, fabric writes the records commit from a temporary index…". Also the new tool fact "`git commit -- <paths>` during a merge fails…".
   - The case: in a one-repository hub, agents work on the code branch, so an agent can leave a `git cherry-pick --no-commit` or a conflicted `git rebase` in the task worktree. `git commit -- <paths>` then dies with "cannot do a partial commit during a cherry-pick." or "…during a rebase." (git's `builtin/commit.c` has these beside the merge message).
   - What breaks: `MergeStateActive` probes only `MERGE_HEAD` and conflicted files (`internal/fabricengine/mergestateactive.go:33-42`). A conflict-free cherry-pick passes the probe and the re-probe, so `failureDisposition` returns the commit error (`internal/statuscommit/statuscommit.go:169-181`) and the run halts. A conflicted one reads as mid-merge, and the run waits for a merge that does not exist.
   - Why it matters: F24's premise, "any sequencer state in the shared worktree breaks transitions", is resolved only for merges. The tool fact pins only the merge message.

2. **MEDIUM: the crash-resume text names a way forward the waiting run cannot reach.**
   - Changed text: D5's new paragraph, "one before step 3 leaves the source's records in the index, so D4 holds records commits until the merge is aborted. The next run of the merge verb aborts the merge and starts again from step 1."
   - The case: a producer crashes between step 2 and step 3. The run's next transition then waits, under the new D4 rule, until the merge is aborted. The merge verb that would abort it runs only in a later producer, which starts only after that transition lands.
   - What breaks: the run deadlocks until the operator runs `lym fabric merge --abort`. The record presents the verb's own re-run as the resume path, and that re-run is unreachable.
   - Evidence: `mergeresolve.Resolver.Resolve` aborts an in-progress merge only when it is called (`internal/mergeresolve/mergeresolve.go:40-48`). The seam waits before the next producer starts (`PATTERN-run-fsm`, "a transition commits and pushes before the run acts").

3. **LOW: the git-only fabric-merge predicate admits a hand-made merge that copies the trailer.**
   - Changed text: D7, "A merge commit fabric wrote is a merge commit carrying fabric's merge trailer whose tree equals its first parent's records paths over the clean merge of its two parents' records-stripped trees."
   - The case: the operator, in the prime, runs a clean `git merge` of a local side branch holding unpushed code commits, and pastes fabric's merge trailer into the message. The tree test passes, because a clean merge of the stripped trees leaves the first parent's records untouched. The guard then admits the merge and, through "a commit such a merge brings in through its second parent", pushes the operator's side-branch code.
   - Evidence: this contradicts D7's own line "A commit message alone never admits a commit that changes code paths". The `.lym` SHA the proposal replaced closed that hole.
   - Severity: this needs deliberate imitation, so it is low.

4. **LOW: squash landings into the prime fall outside the new predicate, and the new Bound names only hand-resolved merges.**
   - Changed text: the D7 Bound, "A merge fabric wrote over hand-resolved code conflicts does not read as a merge commit fabric wrote; the guard owes it as `operator_unpushed`…".
   - The case: D5 includes squash merges. A squash landing into the prime's branch is a one-parent commit that changes code paths, so it is neither records-only nor a merge commit. When the landing's push fails, the next prime records push owes it as `operator_unpushed` too, and the Bound is silent about that.

No further tool calls made.
