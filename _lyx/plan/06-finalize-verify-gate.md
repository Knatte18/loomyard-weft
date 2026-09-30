# Card 6 — Finalize: verify the tree before landing on the parent

**Edit:**
- `internal/landingshed#Finalize`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.mergeInStep`
- `internal/landingshed/doc.go`

**Create:**
- `internal/landingshed/finalize_verify_test.go`

**Uses:**
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`

**Intent:**
Stop `Finalize` from landing on the parent, and pushing it, with a tree its catch-up merge-in broke — the worse case, since nothing reviews it downstream.
Add a `gate verifyGate` field to `Finalize`, set by `NewFinalize` through `newVerifyGate(deps)`; existing struct-literal tests leave it zero-valued (no gate) and keep passing.

Put the gate inside `mergeInStep`, after the `OutcomeStuck` return:
call `fz.gate.check(ctx, finalizeName, fz.deps.ParentBranch, !result.AlreadyUpToDate)`.
A returned error goes through `cancelErr` first and is otherwise returned wrapped with the producer name, with `done=true`;
a non-empty reason returns `fz.stuckOrCancelled(ctx, reason)` with `done=true`.
Because both step 2 and step 5's merge-in-required retry call `mergeInStep`, every merge-in — the first and the retry after the parent moved again — is followed by the gate, and a failure returns before the parent opener, the parent-side merge, `MarkTaskDone`, the parent push and the pull-request close.
Update `mergeInStep`'s doc comment to say so.

Update `internal/landingshed/doc.go`'s Finalize paragraph: each catch-up merge-in runs the same post-merge verify gate as Publish before the parent-side merge, and a failure is Stuck with the parent branch untouched.
Use semantic line breaks in the changed comment lines.

`finalize_verify_test.go` (untagged) builds `Finalize` with `newVerifyGate(deps)`, a fake `run`, and the package's `recordingParentMerger` and `newFinalizeDeps` helpers, with a well-formed summary written.
Assert:
tree changed and fail → Stuck before the parent-side merge (the merger's `Merge` and `PushBranch` never called), a recording `MarkTaskDone` never called, the marker remains;
tree changed and pass → the parent-side merge and push run and the verdict is Done;
already up to date with a marker → the runner runs, fail is Stuck before the parent-side merge, pass removes the marker and lands;
a merge-in-required retry whose second merge-in changed the tree → the gate runs again after that merge-in: script the resolver to return tree-changed on both calls, the fake runner to pass its first run and fail its second, and the merger to return `*fabricengine.ErrMergeInRequired` on its first `Merge`;
assert the runner is called exactly twice, the verdict is Stuck, `Merge` is called exactly once, `PushBranch` and `MarkTaskDone` are never called, and the marker remains.
A resolver that returns a scripted result per call is a small new in-file fake, since `recordingResolver` returns one scripted result.

**ImpactSummary:** `Finalize` gains one field and `mergeInStep` gains the gate, reaching both of `Call`'s merge-in sites; the parent-side merge, board update, push and pull-request close are untouched.

**Commit:** 6: landingshed: verify the tree after Finalize's parent merge-in
