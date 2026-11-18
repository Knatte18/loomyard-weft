```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking every claim the fix added or rewrote against the code at HEAD.

**Finding 1: MEDIUM.** Changed text in D2 says the refusal is recognised "as `protectedBranchRefusal` in `internal/landingshed/finalize.go` recognises it". D7 says the push is one "that origin refuses as a protected branch, as `protectedBranchRefusal` recognises it".
- **Summary:** both decisions put the recognition inside fabric (clone, and the prime records push), but the recogniser is an unexported function of `landingshed`, which fabric does not and may not import. As written, fabric cannot carry out the step.
- **Evidence:**
  - `internal/landingshed/finalize.go:898-909` declares `protectedBranchMarkers` and `func protectedBranchRefusal`, both unexported.
  - No file in `internal/fabricengine/` imports `internal/landingshed`.
  - `landingshed` imports fabricengine (`internal/landingshed/publish_integration_test.go:51`), so an import the other way would make a cycle.
  - The record should say the marker match moves to fabric, or to `gitrepo` beside push classification, and that `landingshed` reads it from there.

**Finding 2: LOW.** Changed text in D7: "its failed-push record has class `deferred`, which the act rule already reads as owed".
- **Summary:** the act rule does read `deferred` as owed. But in the code `deferred` means a push that was held back and that the next push of the branch retries. The guard's refusal is retried by every later records push and refused again until the operator pushes. That is a new meaning for the class, and the record does not state it.
- **Evidence:**
  - `internal/fabricengine/failedpush.go:39`: "PushFailureDeferred means the push was held back, not refused, and is retried by the next push of the branch."
  - `FailedPushesOwedBy` reads `deferred` as owed (`internal/fabricengine/failedpush.go:272-293`).

These claims were checked and hold:
- `TestCommitPathAllowEmpty`'s row `UnchangedPathWithAnotherPathStaged` pins `git commit … -- &lt;path&gt;` leaving other staged entries staged (`internal/gitrepo/gitrepo.go:242` runs `commit --allow-empty -m msg -- relPath`).
- `shedrun.RunsRootRel` exists (`internal/shedrun/paths.go:483`).
- `fabricengine.Fabric.ReadParentTips` exists (`internal/fabricengine/parenttips.go:47`).
- `fabricengine.ReturnToPreviousTransition` and `deleteGivenUpCodeBranch` are where D6 says they are.
- `.lym` is untracked and excluded, so a checkout does not change it.

Tool calls beyond my context:
- Read the failed-push class comments, `RunsRootRel`, the `ReadParentTips` signature, the `CommitPathAllowEmpty` body and where `protectedBranchRefusal` is declared. My context held none of these.
- Checked where `RunsRootRel` is defined and whether fabricengine imports `landingshed`. The first grep's pattern had missed `RunsRootRel`, and the import direction was not in my context.
