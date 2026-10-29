```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths, on the decision record for pr-review-row: I traced the code paths that already read or write each decision's state, and two decisions break on existing paths.

**MEDIUM — D8 (D6, D9): the agent's ungated commits can reach the pull request without the row's gate.**
- **Why it breaks:** `Fabric.Commit` fires a detached push of both sides whenever run records are committed in the task pair. That push loops over "whichever side has commits", so it carries the task branch along. D6's own record write, a status commit, or any `lym` command the agent runs that commits records will push its local commits before the `pr-review` gate has judged them. The pull request then updates with code the gate never saw.
- **What still holds:** D9's Done rule keys on the record's gated head, so a stray push does not land anything.
- **What breaks:** D8's promise that no head reaches the pull request ungated.
- **What the record needs:** say how agent commits stay off origin until gated, or narrow the bound to "Done reads only gated heads".
- **Evidence:** `internal/fabricengine/doc.go` near line 244 (the detached both-sides push, its coalescing loop that pushes whichever side has commits, and the note that `SkipPush` does not stop it).

**MEDIUM — D12: only `lym batten cancel` closes the pull request, but a run is torn down by other routes too.**
- **Why it breaks:** these packages also call the pair teardown:
  - `cleanupcli/reconcile.go`, which tears down runs origin reports done or removed, for example a board removal on another machine;
  - `fabriccli/remove.go` (`lym fabric remove`);
  - `adoptcli`, `hibernatecli/reconcile.go`, `battencli/resumereconcile.go`, `battencli/beamfork.go` and `loomcli/ownercheck.go`.
- **Result:** any of these leaves the pull request open while the record is removed, so nothing can close it afterwards. Meanwhile `runproc.EndSparing` ends the agent strand, which survives fine.
- **What the record needs:** hook the close into the shared teardown, or state the gap as a bound.
- **Evidence:** the grep of teardown callers above.

**LOW — D6, D8: the push the row uses must be the blocking one.**
- **Why it breaks:** "the run's existing push route" is, by default, the fire-and-forget detached push. Opening a pull request, and recording "the head it last pushed", right after an async push can race: GitHub may not have the head branch yet, or the record may name a head origin lacks. `CoalescePushBothAt` is the blocking form.
- **Evidence:** `internal/fabricengine/coalesce.go` line 79.

**LOW — D10 against D8: the verify inside the wait erases the `PR #&lt;n&gt;` mark.**
- **Why it breaks:** the driver strand's wait mark is set and cleared through `driverWaitMark`, the `VerifyWaitMark` seam. Each `pr-review` gate run inside the wait marks the driver strand `verify PR-Review` and clears the mark on its way out, which also removes D10's mark. The record should restore D10's mark after each gate run.
- **Evidence:** `internal/loomcli/sharedbootstrap.go` line 330; landingshed's `VerifyWaitMark` paragraph clears "on every way out of the verify".

Paths traced, by decision:
- **D1:** I traced reads with the parent's worktree as anchor, the configengine local rule, run_settings seeding (closed to a cross-pair key) and Finalize not carrying per-worktree config to the parent. All survive.
- **D2:** I traced the board's tolerant unknown-key handling for older binaries and `upsert` validation. Both survive.
- **D3:** I traced the retired-key load and prune and the entry gate's Skipped history. Both survive.
- **D5:** I traced the githubclient cache path. It survives provided each named account gets its own cache file. One shared file would read the other account's token as a foreign shape and swap the two back and forth.
- **D6, D9:** I traced the origin reconcile's "ended" verdict. `readEndedSet` reads the board status on Bolt, not the code, so a GitHub merge does not mark the run ended before Finalize, and D9's Done survives. The reconcile's run-scope halt also survives.
- **D7:** I traced the interrupt-reinvoke re-entry and teardown's `runproc.EndSparing` ending the strand. Both survive.
- **D9:** the parent lock's idle release applies only to lock holders, and PR-Review holds none, so it survives. Batten InnerRun yields on a running child, which also survives.
- **D11:**
  - The sync's fast-forward survives.
  - The `HoldsWork` merge-tree test recognises a squash, so the already-landed path survives.
  - `deleteTaskBranchOnOrigin` treats an absent remote branch as no reason, so GitHub auto-deleting the head branch survives.
  - InnerRun's owner check names `finalizeRow`, which is unchanged, and survives.
- **D8:** breaks on the detached push, as in the first finding.
- **D12:** breaks on the other teardown routes, as in the second finding.
- **D10:** the wait mark breaks on the verify's clear, as in the last LOW finding.
- **D13:** the pawl hold-free rows stay Beam-Up and Beam-Await, and the new row sits after Parent-Align, so it survives.

Tool calls I made:
1. Grepped fabricengine's ended-set functions and the `runproc` End callers. I needed to know how origin "ended" is judged and which paths end the agent's processes.
2. Grepped the Finalize row-name readers and the push entry points. I needed to know what reads the Finalize row by name and how code-branch pushes are triggered.
3. Read `internal/fabricengine/doc.go` lines 244–262, the start of `internal/battenshed/innerrun_owner.go` and the idle notes in parentlock's doc. These give the detached push's behaviour, the owner check and the lock's idle release.
4. Grepped the pair-teardown callers and the `SetWaitMark` users. I needed every teardown route for D12 and every writer of the driver's wait mark for D10.
