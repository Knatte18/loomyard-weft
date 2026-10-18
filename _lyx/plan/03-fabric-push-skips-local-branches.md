# Card 3 — fabric-push-skips-local-branches

**Edit:**
- `internal/fabricengine#PushResult`
- `internal/fabricengine#PushAnchored`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#unpushedOutcome`
- `internal/fabricengine_test#TestPushAnchored_PushesAndRecordsBranchPush`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestCoalescePushBothAt_AdvancesBothSidesAndLeavesNoWarpRootLock`

**Uses:**
- `plan:internal/gitrepo#ErrLocalBranch`
- `plan:internal/gitrepo#Repo.MarkBranchLocal`

**Intent:** Make fabric's synchronous pushes and the detached child's push report a local-only branch as skipped rather than failed (D3: "`PushAnchored`, `PushPairAnchored` and `lyx fabric push` skip a marked branch and report it skipped on their results"; "The detached child ... skips a marked branch through the same check and logs it").

`PushResult` gains `LocalSkipped []string` (`json:"local_skipped,omitempty"`), the branch names a push left unpublished because they carry the local mark.
`PushAnchored` and each side of `PushPairAnchored` map a push error satisfying `errors.Is(err, gitrepo.ErrLocalBranch)` to an entry in `LocalSkipped` naming that side's current branch and a nil error for that side, recording no `KindBranchPushed`; any other error keeps today's handling, so a rejection still surfaces `gitrepo.ErrPushRejected` as before.
`unpushedOutcome`, the mapping `CoalescePushBothAt`'s steps use, maps `ErrLocalBranch` to an Info log naming the path ("fabricengine: local-only branch, not pushed") and nil, so the detached child logs the skip and its loop sees no progress on that side.
What this lets through: a push call on a marked branch now succeeds having published nothing; what bounds it: only `ErrLocalBranch` is mapped, which card 2's mark alone produces.

`lyx fabric push`'s envelope key for `LocalSkipped` lands with card 4, whose scenario is the first that holds a local pair.

Tests, each a step or row in the existing test over the hubforge hub it already builds:
- `TestPushAnchored_PushesAndRecordsBranchPush`: after the existing push, mark the records branch local with `gitrepo.Repo.MarkBranchLocal`, commit one more records file, and `PushAnchored` returns nil, `LocalSkipped` naming the branch, no `branch_pushed` entry, and the remote tip unchanged.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`: a row marking the pair's code branch local pushes the records side, reports the code branch in `LocalSkipped`, and leaves the code branch's remote tip unchanged.
- `TestCoalescePushBothAt_AdvancesBothSidesAndLeavesNoWarpRootLock`: a step marking the warp branch local, then committing on both sides, ends nil with the weft side pushed and the warp remote tip unchanged.

**ImpactSummary:** Three fabricengine push entries gain one mapped error and one result field; callers reading `PushResult` see a new optional field and no changed error.

**Commit:** 3: fabricengine: pushes report local-only branches as skipped
