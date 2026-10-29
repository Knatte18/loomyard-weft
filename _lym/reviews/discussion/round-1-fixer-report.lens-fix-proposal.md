```yaml
lens: fix-proposal
outcome: reported
```

**fix-proposal lens: challenging the reviewer's proposals the fixer carried out**

Fixes checked: F1, F2, F3 (residual-bound option), F5, F6, F7, F8, F9, F10, F11, F12 (bound option) and F13.

1. **MEDIUM. F5 left the local parent behind origin when Finalize lands through the already-landed path.**
   - Changed text: D11's fourth bullet now reads "It then fetches the parent and lands through Finalize's existing already-landed path…". The D11 acceptance criterion changed with it.
   - Summary: Dropping the fast-forward means the already-landed path is not detected. Finalize would squash the task again onto a parent that is behind origin.
   - Evidence: `fabricengine.ParentHoldsTaskWork` (`internal/fabricengine/replay.go:312`) checks whether the **local** parent branch holds the task's work. Finalize's comment says the parent is fast-forwarded to its upstream only in `syncParent`, which runs earlier in the same call, before the GitHub merge.
   - The case: after the GitHub merge and a bare fetch, the local parent does not hold the work. Fabric.Merge then commits a fresh squash on the stale local parent, and the parent push is rejected.
   - Needed: keep the fast-forward of the local parent pair to its upstream before handing off to the already-landed path. Keep it in the criterion too.

2. **MEDIUM. F3's residual bound lets the agent stand in for the operator's approval.**
   - Changed text: D7's Bound, "An approval it gives with an account other than the pull request's author counts as a standing approval…".
   - Summary: In the setup D5 exists for, the session can approve the pull request itself, and D9 then ends the run Done without the operator.
   - Evidence: D5's Rationale says the session's environment token belongs to the session's account, which differs from `pr_account` on Models. So the session's `gh pr review --approve` is not an author approval, and D9's approval bullet ends Done.
   - The `finalize` gate only checks the tree, not who approved. The reviewer's other option, a deny on the launch spec, would not leave this gap.

3. **MEDIUM. The new D7 acceptance criterion contradicts itself.**
   - Changed text: "a head the session pushes is gated by the row before anything lands it, and a merge it makes of an ungated head is D9's ungated-merge halt".
   - Summary: A merge the session makes on GitHub lands an ungated head on the parent before the row sees it. The halt fires afterwards, so "before anything lands it" is false.
   - Evidence: D9's first merged bullet halts only after the merge, and its way forward checks the parent by hand.

4. **MEDIUM. The way forward in F2's ungated-merge halt never marks the board task done, though the work is on the parent.**
   - Changed text: D9 bullet "merged, with a merged head other than the record's gated head: … `lym batten cancel <slug>` as the way forward".
   - Summary: Cancel tears the pair down unlanded, so the board entry is not closed even though GitHub merged the work.
   - Evidence: in the same D11, the "merged on GitHub by someone else" case goes through Finalize's already-landed path, which marks the task done. The new halt has no route there.
   - The case: the operator accepts a review suggestion in the GitHub web interface (a commit the task branch lacks), then merges. The run halts and can only be cancelled, leaving a done task looking open.

5. **LOW. One of the two ways forward in F7's dirty-tree halt only leads to another halt.**
   - Changed text: D9 bullet "merged, with the merged head equal to the task head, on a dirty tree: … committing or discarding them in the task worktree, then `lym loom resume`".
   - Summary: Committing creates task commits the merged head lacks. The resumed call then hits the lacking-commits halt, which ends only in cancel. Only discarding reaches Done.
   - Evidence: D9's lacking-commits bullet.
   - The resumed call also restarts the stopped session (D7: "one that finds the recorded session stopped resumes it"), only for the merged branch to stop it again.

Tool calls:
- One grep for `ParentHoldsTaskWork`, to learn whether it reads the local or the remote parent, which my context did not show.
