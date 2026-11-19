```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: I traced each guard, refusal, branch and test row the plan adds, to check its state can occur from a real entry point and that no guarded site recurs unguarded.

**BLOCKING — Card 9, `TestRemove_NeverPushedRecordsBranchRecordsNoRemoteDeletion` (D4): the row's expected state cannot occur, so the test fails in both layouts.**
The test expects a pair added with no push and removed with its remote step to record no `remote_branch_deleted`. But `Topology.Remove`'s remote step pushes the unpushed weft tip first, then deletes it with a lease, and records the deletion.
- `internal/fabricengine/remoteteardown.go:127` calls `archiveWeftTip(..., job.Kind == remoteTeardownRemove, ...)`, so `pushBranch` is true on remove.
- In `internal/fabricengine#archiveWeftTip` (archive.go:~150-158), when origin lacks the tip it runs `PushCommit(origin, tip, weftBranch)` and sets `originTip = tip`, so `leaseTip = tip`.
- `remoteteardown.go:131-139` then runs `deleteWeftBranchOnRemote(..., out.archive.leaseTip, ...)`. It reaches `deleteRemoteBranch` with that lease (cleanup.go:425-447), which appends `KindRemoteBranchDeleted` (destroy.go:1617-1626).
- Card 3's ls-remote probe inside the plain `DeleteRemoteBranch` is never on this path.

**MEDIUM — Card 3, the `DeleteRemoteBranch` ls-remote probe (D4): no production entry point reaches the guarded state.**
Every production caller of `internal/fabricengine#deleteRemoteBranch` passes a `leaseSHA`, which routes to `DeleteRemoteBranchLeased`:
- cleanup.go:425-447 returns early when `leaseSHA == ""`;
- add.go:735 leases on `weftOld.tip`;
- remove.go:693 leases on the observed tip;
- handover.go:332 leases on the tip.
The plain delete, where the probe tells "absent" from "removed", is therefore reached only by `TestDeleteRemoteBranch`. D4's "never-pushed teardown records no deletion" runs through the leased archive path above, which the plan leaves as it is.

**MEDIUM — Card 5, join path: the refusal of a binding that names another URL cannot be reached as written.**
The card resolves the join's `.lym-warp` "through `resolveEffectiveWarpURL` … as the probe's answers are". The probe path passes an empty `supplied` (clone.go:194). With `found && supplied == ""`, `internal/fabricengine#resolveEffectiveWarpURL` returns the recorded URL and never refuses (codebinding.go:156-157). The mismatch refusal fires only if the join passes the clone URL as `supplied`. No card 5 or card 7 row reaches it either.

**LOW — Card 5/7, join path: the wrong-`--shortname` refusal has no reaching test.**
`resolveEffectiveShortname` refuses a mismatching `--shortname` on the join, and the card says the hub is then torn down. `TestCloneOneURL_JoinsTheNamespacedHub` has no row with a wrong shortname.

**LOW — Card 5, seed path: the observed records tip is keyed on origin's `HEAD` symref, but the prime branch comes from the code clone.**
`cloneSeed`'s observed tip is read from `originLayout` under origin's default branch, taken from the `HEAD` symref line. The records branch pushed is `RecordsBranchName(checkedOutBranch(prime))` (clone.go:296). Over an origin whose `HEAD` is unborn, `ls-remote --symref` may name no default branch. The prime branch then comes from local `init.defaultBranch`, so the lease could be taken on the wrong ref or on absence. The card states no rule for that case, and no test row covers an empty origin.

Guards checked and found reachable, with no unguarded recurrence:
- Card 5's `lym-bolt`, board-present, `.lym-warp`, root-`.lym-anchor`/`_lym` and second-URL refusals.
- Card 5's two-URL `SingleRepo` offline refusal, through card 6's tier0 row.
- Card 5's refused, lease-lost and restore branches, through card 7's hook and leftover rows.
- Card 8's unset-key refusal.
- Card 2's missing-path, no-origin and error rows.
- Card 3's namespaced retry row.
- Card 4's listings: `lsRemoteHeads`, `fetchMovedHeads` and the board target read the namespace at their own `dir`. The other ls-remote hits (refscanner.go:557, remove.go:222, originreconcile.go:764/784/850) are a flag table or comments, not records-store calls.

Tool calls beyond my context:
- Read remove.go:640-700, cleanup.go:425-450 and the unlisted ls-remote sites, to find which delete form the teardown uses and whether those sites send origin refs.
- Read `resolveEffectiveWarpURL` and the `readBranch` call sites, to check the join's binding refusal and other places that read the board's branch.
- Grepped remoteteardown.go, remove.go and add.go for `deleteRemoteBranch`, `leaseSHA` and `DeleteWeftOnOrigin`, to trace the weft delete on teardown.
- Read remoteteardown.go:125-150, to see how the weft delete is leased.
- Read archive.go:100-180 and handover.go's delete, to confirm `archiveWeftTip` pushes the weft tip before the leased delete, and that handover's delete is leased.
