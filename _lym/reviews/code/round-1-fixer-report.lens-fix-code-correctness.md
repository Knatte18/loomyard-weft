```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code correctness (F1–F5) through the correctness lens.

Findings:

1. **MEDIUM**, `internal/beamcli/deps.go:316`, the new `pairteardown.PushRun(prime, beam, slug, RunPush{LockWait: …})` in `PushBeam`.
   - Summary: `PushRun` now removes the local beam pair on a takeover, and `PushBeam` never says whether the caller is inside that pair.
   - Evidence:
     - `pairteardown/handover.go:68` and `:115` show that without `InsidePair` a hand-over ends the run's processes and removes the local pair.
     - Every other `PushRun` call made from within a run's pair sets `InsidePair: true` (`loomcli/wiring.go:225`, `loomcli/landingdeps.go:416`, `:484`, `:953`).
     - `PushBeam` is reached from `beamcli` verbs (`pr.go:81`, `:172`) that a user can run from a pair.
     - `TestLand_RunsFromATaskPair` at `land_integration_test.go:24` exercises a land from a task pair.
     - A rejected beam push over another machine's takeover could therefore delete the worktree the verb runs in.
     - The previous code only returned a raw error and removed nothing.
   - Suggested change: decide `InsidePair` from the caller's location.
   - A second, smaller gap: the review asked for a test row showing a rejected beam push over another machine's takeover hands over, and none was added, so the changed branch is unexercised. This follows from the fixer's own choice not to add one.

2. **LOW**, `internal/fabricengine/handover.go` `pushOwedCodeTip`, the `originAbsent` branch.
   - Summary: an absent origin branch is now always recreated, including a task branch origin deleted on purpose.
   - Evidence:
     - `remoteHeadTip` cannot tell "never pushed" from "deleted after a landing".
     - The code gates only on `lastTransitionCodeTip` being non-empty and `holdsCommit`.
     - A takeover entry implies the run is still live, so the case is narrow, and I found no code that blocks it.
   - Suggested change: none required. Reported only so the fixer can confirm the case is out of scope.

Checked and correct:

- **F1, push:**
  - With an absent origin, `PushCommit(tip, branch)` is non-forcing and creates the branch.
  - `rec.AppendRef` records it.
  - `owedTipPushedHook` and the records re-read still run, because they follow the `if/else`.
- **F1, give-up:**
  - `deleteGivenUpCodeBranch` calls `createTagAt`, then `pushArchiveTag`, then `deleteRemoteBranch`.
  - `checkRemoteBranchRequest` passes with `dirtyArchivedOnRemote(tag)` and a non-empty lease: the tag is non-empty and the lease is the tip.
  - The `ownedPairWarpBranch(branch, "")` ownership check passes.
  - The later `archiveOwnerTip` re-tags the same tip, which `createTagAt` reuses, so the repeat is harmless.
- **F1, tests:** both new rows would fail with the defect in place.
  - Row 1 would see `TipPushed` false and no branch.
  - Row 2 would see the branch left alone and no tag.
  - Both ran in the tier-1 log, which matched 4 lines for their names.
- **F3:**
  - `fz.recordLanding(parentHandle)` runs only after `archiveHeldParentMerge` returns true.
  - `recordLandedSize` re-records from `parentHeadMerged` to the reset head, which stays consistent.
  - Other `Finalize` tests are unaffected, because `recordLanding` returns early when both seams are nil.
  - The edited row fails without the fix, since the second `record originTip` entry would be missing.
- **F4 and F5:** these only add comment lines. The `//testtiming:keep` lines sit after a blank `//` line directly above the `func`, as the existing marker at `finalize_test.go:348` does.

Calls made:
- `grep PushRun/InsidePair` across `internal`, plus a read of `land_integration_test.go:20-50`: my context did not show how other `PushRun` callers set `InsidePair`, nor which verbs run from a pair.
