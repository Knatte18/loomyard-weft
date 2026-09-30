---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/08-producer-transport-errors.md:16
    summary: Card 8 still omits Publish.checkApproval's FindPullRequest arm, so a 5xx or timeout on the approve-then-start path persists blocked instead of a github-api failure
  - id: F2
    severity: LOW
    location: _lyx/plan/08-producer-transport-errors.md:19
    summary: Card 8 places the new error branch after the arm's existing cancellation check, but none of the edited arms has one; cancelErr lives inside stuckOrCancelled, which the new branch bypasses
  - id: F3
    severity: LOW
    location: _lyx/plan/08-producer-transport-errors.md:30
    summary: Batten's Worktree-Teardown remote branch deletion is a step-reachable git push whose failure can never be marked, and card 8's out-of-scope list does not name it
---

### [BLOCKING] Card 8 omits Publish's approval-path pull-request query

**Location:** `_lyx/plan/08-producer-transport-errors.md:16`, and the "four arms" count at lines 19 and 46.

**Issue:**
`Publish.Call` makes two `FindPullRequest` calls, not one.
Step 3a runs `checkApproval` (`internal/landingshed/publish.go:116`) whenever `ApprovalPath` and `TaskHead` are set.
When an approval record exists, `checkApproval` queries the pull request at `publish.go:248`.
A failure there goes through `p.stuckOrCancelled(ctx, publishGitHubErrorReason("query existing pull request", err))` and returns `decided: true`, before any merge-in, push or step-7 query.
Card 8 lists only "its pull-request query" (the one at `publish.go:167`), the push, the create and Finalize's parent push, and it says "each of those four arms".
The approval-path query is the call every run makes after `lyx loom approve`, on both the hub's `approve` then `start` route and batten's `Spawn` route, which the decision record's batten acceptance criterion names.
With the plan as written, a GitHub 503 or a `publishGitHubTimeout` deadline on that query still persists `blocked`.
That misses the Goal ("clears the stops that a single re-step would clear") and D2's `github-api` class on the call most likely to follow a hand-back.
None of the plan's tests would catch the gap: the approval tests (`TestPublish_MatchingApproval_DoneWithoutSync` and its siblings) never fail the list call.

**Fix:**
Card 8's Intent names the approval-path query as a fifth arm.
It gets the same treatment as the others: after `cancelErr`, it returns a `%w`-wrapped hard error (`landingshed: Publish: query existing pull request for the approval: %w`) from `checkApproval` with `decided: true`, when `shedtransient.Class` classifies the failure.
It keeps its warn line on both branches.
Add a fake-based test: an approval record on disk, a `TaskHead`, and a list call answering 503, which returns an error classified `github-api` and makes no merge-in or push.
Update the arm counts at lines 19 and 46 to five.

### [LOW] The new error branch must call cancelErr first

**Location:** `_lyx/plan/08-producer-transport-errors.md:19`.

**Issue:**
The card puts the classified-error branch "after the arm's existing cancellation check".
The arms it edits have no check of their own: the step-5 push, the step-7 query, the create, the approval query and Finalize's `pushParent` each call `stuckOrCancelled` directly, and that is where `cancelErr` runs.
A new branch placed before `stuckOrCancelled` skips it.
So a push or GitHub call that fails with a transport error just as the step is cancelled would return the classified error instead of the cancellation.
This breaks `ctx.go`'s rule that every non-Done return consults `cancelErr` first.
It also breaks card 3's premise that "every landingshed arm checks its own context and returns the cancellation before any GitHub error".
The observable damage is small: `shedengine`'s `callErr != nil && ctx.Err() != nil` arm (`internal/shedengine/run.go:230`) persists `paused` for any error under a cancelled context, so the step still pauses.
Only the returned error text and the landingshed convention are affected, which is why this is LOW.

**Fix:**
State that in every arm it edits, the new branch first calls `cancelErr(ctx, <row>)` and returns that error when it is non-nil.
Only then does it test `shedtransient.Class`.
Add one test: a `PushBranch` transport error under an already-cancelled context returns an error wrapping `context.Canceled`, not one classified `git-transport`.

### [LOW] Batten's Worktree-Teardown remote deletion is neither covered nor excluded

**Location:** `_lyx/plan/08-producer-transport-errors.md:30` (the out-of-scope list).

**Issue:**
The second focus question asks whether any other step-path producer makes a remote git or GitHub call that card 8 neither covers nor excludes.
I checked every package that pushes, fetches or calls GitHub.
`fabricengine.Merge`'s fetches (`mergeguards.go:55`, `merge.go:553`) only warn, as do the status-commit pushes (`loomcli/wiring.go:200`, `battencli/commitstatus.go:178`), selfreport's issue filing and Finalize's close calls.
Worktree-Create is excluded with a reason.
One remote call is left over, in batten's Worktree-Teardown row (`internal/battencli/wire.go`), and card 8 says nothing about it.
`top.Remove(location, slug, false, true)` deletes the other-side branch on the remote with `git push --delete`, which is a push under D2.
`Remove` reports a failed deletion as the string `res.RemoteBranchError`, which `fabricengine/weftwiring.go:273` builds with `fmt.Sprintf`.
The row then returns `fmt.Errorf("... remote copy was not deleted: %s; resume this run to retry the deletion", ...)`.
This failure is a hard error, not `Stuck`, but it has no `*GitError` in its chain, so `shedtransient.Class` always returns empty.
Yet a re-step is exactly what the row asks for, since its already-gone arm retries the deletion through `RemovePairBranch`.
The out-of-scope list claims to stop a later reader from widening card 8 by accident, and it names batten's other remote rows but not this one.

**Fix:**
Add an out-of-scope entry to card 8.
Batten's Worktree-Teardown returns a failed remote branch deletion as an unmarked error: `fabricengine.Remove` reports it as text (`RemoveResult.RemoteBranchError`), so there is no error chain to classify, and giving it one would widen `fabricengine`'s result beyond the decision record's scope.
The driver's existing trace-based repair path and hand-back still handle it.

## Process note

I read `round-1-fixer-report.md` and `round-1-bouncer-verdict.md`, and grepped the headings of `round-1-review.md`, during exploration.
That was before this round's instructions said to wait until the review was saved.
The focus directive's reference to round 1's BLOCKING finding prompted it.
F1 and F2 match two of the bouncer's round-1 new findings, and I confirmed both independently against `internal/landingshed/publish.go`, `finalize.go`, `ctx.go` and `internal/shedengine/run.go`.
The plan's card files have not changed since the round-1 fixer wrote them.
F3 is new this round.
