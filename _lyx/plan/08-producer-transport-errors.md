# Card 8 — landing producers return transient failures as errors

**Edit:**
- `internal/landingshed/publish.go`
- `internal/landingshed/publish_test.go`
- `internal/landingshed/finalize.go`
- `internal/landingshed/finalize_test.go`
- `internal/landingshed/seam_enforcement_test.go`

**Uses:**
- `internal/shedtransient#Class`

**Intent:**
Make `git-transport` and `github-api` reachable in a real run (decision record D1, D2).
The step-reachable remote calls sit in the landing rows, and today every one of their failures ends as a verdict, never as the producer error that card 7's classifier sees:
Publish's task-branch push, its step-7 pull-request query and its pull-request create each return `stuckOrCancelled`, and so does Finalize's parent-branch push.
So does Publish's second pull-request query, in `checkApproval`, which step 3a runs whenever an approval record exists: it is the first GitHub call after every `lyx loom approve`, on both the hub's `approve` then `start` route and batten's `Spawn` route.
Neither row has an `on_stuck`, so each persists `blocked` and hands back with no transient re-step.

None of these five arms checks its context before calling `stuckOrCancelled`, which is where `cancelErr` runs today.
So in each of them the new branch first calls `cancelErr(ctx, <row>)` and returns that error when it is non-nil, keeping `ctx.go`'s rule that every non-Done return consults `cancelErr` first.
Only then, a failure for which `shedtransient.Class` returns a non-empty class is returned as a hard error wrapped with `%w`, in the form `landingshed: <row>: <action>: %w`.
In `checkApproval` that error is returned with `decided: true`, so `Call` returns it before any merge-in or push, and its action reads `query existing pull request for the approval`.
The `Shed.Transient` classifier then marks it, and the step persists `failed` with the class.
Every other failure keeps today's `Stuck` verdict, reason text and warn log, unchanged:
`gitrepo.ErrPushRejected`, an authentication failure, a 4xx or rate-limited GitHub response, an unresolvable token, and an unusable origin URL.
The GitHub arms keep their existing warn line on both branches.
The doc comments of `Publish.Call`, `checkApproval`, `Finalize`'s parent-push step and `stuckOrCancelled` state the split: a transient failure is an error, so the driver re-steps once, and anything else is a verdict for a human.

A re-step after such an error re-enters the row from its start, which both rows already tolerate.
Publish re-runs its approval check, merge-in and push and then queries for an existing pull request before creating one, so a create that reached GitHub before its 5xx is found, not duplicated.
Finalize's parent-side `Merge` reports `AlreadyUpToDate` for a branch it already merged (`TestFinalize_AlreadyUpToDateMergeProceedsAsLanded`), so the re-step goes straight to the parent push.

Out of scope, stated in `Publish.Call`'s doc comment so a later reader does not widen it by accident:
- `Finalize`'s pull-request close calls only warn and never block, so they stay as they are.
- Batten's Worktree-Create row also turns a failed `fabricengine.Add` push into `Stuck`.
  It stays a verdict, because `Add`'s rollback keeps the branch it made, so an immediate re-step would stop again on the leftover branch rather than clear.
- The status-commit push and batten's Seed-Child push only warn, by design.
- Batten's Worktree-Teardown returns a failed remote branch deletion as an unmarked error.
  `fabricengine.Remove` reports that failure as text (`RemoveResult.RemoteBranchError`), so the error has no chain to classify, and giving it one would widen `fabricengine`'s result beyond the decision record's scope.
  The driver's trace-based repair path and hand-back still handle it.

`landingshed`'s import allowlist (`seam_enforcement_test.go`) admits `internal/shedtransient`, with a one-line reason beside the entry.

Tests (untagged, fakes only):
- `publish_test.go`:
  - A `PushBranch` returning `fmt.Errorf("gitrepo: git push: %w", &gitexec.GitError{Args: []string{"-c", "push.autoSetupRemote=true", "push"}, ExitCode: 128, Stderr: "fatal: unable to access '...': Failed to connect to 127.0.0.1 port 1"})` returns a non-nil error whose `shedtransient.Class` is `git-transport`, and makes no GitHub call.
  - `TestPublish_PushFails_NoGitHubCall` (a plain error) and `TestPublish_PushRejected_DistinctReason` keep their `Stuck` expectations.
  - A pull-request list answering 503 and a create answering 502 each return an error classified `github-api`, and still log the warn line with its action, owner, repo and cause fields.
  - `TestPublish_QueryExistingPRFails_WarnsWithActionOwnerRepoAndCause` and `TestPublish_CreatePRFails_WarnsWithActionOwnerRepoAndCause` switch their fake status from 500 to 422, so they keep covering the `Stuck`-and-warn branch.
  - With an approval record on disk and a `TaskHead`, a pull-request list answering 503 returns an error classified `github-api`, logs the warn line, and runs no merge-in and no push; the same list answering 422 keeps today's `Stuck`.
  - A `PushBranch` returning the transport `*GitError` under an already-cancelled context returns an error wrapping `context.Canceled` whose `shedtransient.Class` is empty.
- `finalize_test.go`: a parent `PushBranch` returning the same transport `*GitError` returns an error classified `git-transport`; a plain push error keeps today's `Stuck` reason naming the manual push.

**ImpactSummary:** Five failure arms in two landing producers now return a hard error for a transient-classified failure instead of `blocked`, and two existing Publish tests move from a 500 to a 422 fixture; every other failure keeps its verdict and text, and no recipe row, batten producer or caller outside `landingshed` changes.

**Commit:** 8: landingshed: return transient push and GitHub failures as errors
