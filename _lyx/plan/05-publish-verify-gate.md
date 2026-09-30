# Card 5 — Publish: verify the tree before pushing

**Edit:**
- `internal/landingshed#Publish`
- `internal/landingshed#NewPublish`
- `internal/landingshed#Publish.Call`
- `internal/landingshed/doc.go`

**Create:**
- `internal/landingshed/publish_verify_test.go`

**Uses:**
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`

**Intent:**
Stop `Publish` from pushing a tree a parent merge-in broke.
Add a `gate verifyGate` field to `Publish`, set by `NewPublish` through `newVerifyGate(deps)`.
Existing tests that build `&Publish{deps: deps, resolver: res}` leave the gate zero-valued, whose nil command is the no-gate case, so they keep passing unchanged.

In `Publish.Call`, insert a new step between step 4 (the merge-in and its `OutcomeStuck` return) and step 5 (the push):
call `p.gate.check(ctx, publishName, p.deps.ParentBranch, !mergeResult.AlreadyUpToDate)`.
A returned error goes through `cancelErr` first and is otherwise returned wrapped with the producer name, never Stuck;
a non-empty reason returns through `p.stuckOrCancelled(ctx, reason)`, so no push, no owner/repo parse and no GitHub client is ever reached.
Comment the step with why it exists: a merge can compile cleanly and still break tests, and only a no-op merge leaves the tree the plan-level verify already passed.
The no-PR-required early return, the push-skipped refusal and the approval path all return before the merge-in, so they run no verify — leave them untouched.

Update `internal/landingshed/doc.go`'s Publish paragraph: after the merge-in, a merge that changed the task tree, or a pending-verify marker left by an earlier unverified merge, runs the told verify command before the push;
a failure is Stuck with the merge commit kept for the operator to fix forward, and a missing command logs a warning and proceeds.
Use semantic line breaks in the changed comment lines.

`publish_verify_test.go` (untagged) builds `Publish` with `newVerifyGate(deps)` and a fake `run`, and reuses the package's `recordingResolver`, `newTestDeps`, `writeSummary` and `newPublishGitHubServer` helpers.
Assert, per acceptance criterion:
tree changed and pass → the push closure runs, the pull request is created (Awaiting), and the marker is absent;
tree changed and fail → Stuck whose reason names the parent branch, the exit code and the output path, the push closure never runs, `NewGitHubClient` is never called (install a factory that fails the test if called), the marker remains, and the output file holds the line the fake runner wrote into its writer;
already up to date with no marker → the fake runner is never called and the push runs;
already up to date with a marker → the runner is called, pass removes the marker and pushes, fail is Stuck with no push;
resolver `OutcomeStuck` → Stuck with the resolver's reason, the runner never called and no marker written;
empty command → the push runs, no marker is written, and a `WARN` line is captured;
empty command with a marker → the push runs, the `WARN` line names the marker path, and the marker stays;
closure error → a returned error, not Stuck, and no push;
tree changed and a marker write failure (the marker path under a regular file, so its parent cannot be created) → a returned error, not Stuck, no push, and the runner never called;
spawn failure → Stuck carrying the spawn error, no push, marker remains;
a context cancelled by the fake runner → a returned error, not Stuck, and the marker remains;
the no-PR-required path and the approval path (reuse `newApprovalFixture`) → the runner is never called.

**ImpactSummary:** `Publish` gains one field and one step between merge-in and push; `Publish.Call`'s other steps, the approval path and every existing test are unaffected.

**Commit:** 5: landingshed: verify the tree after Publish's parent merge-in
