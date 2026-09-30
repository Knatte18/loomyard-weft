# Plan-Review round 2 — fixer report

I fixed all three findings, all in `_lyx/plan/08-producer-transport-errors.md`.
No other card changed.
`lyx loom validate-plan` reports `ok: true` with only informational `create-new-unit` findings.

## Changes

- **F1** (approval-path query arm): card 8's Intent names `checkApproval`'s `FindPullRequest` as a fifth arm, the first GitHub call after every `lyx loom approve`.
  A classified failure there returns a `%w`-wrapped hard error with `decided: true`, so `Call` returns it before any merge-in or push.
  The error's action reads `query existing pull request for the approval`.
  The re-entry paragraph now says a re-step re-runs the approval check first.
  `checkApproval`'s doc comment joins the list of comments that state the error-versus-verdict split.
  A new test: with an approval record and a `TaskHead`, a 503 on the list call returns an error classified `github-api`, and a 422 keeps `Stuck`.
  The ImpactSummary now counts five arms.
- **F2** (cancellation first): the card no longer mentions "the arm's existing cancellation check".
  It says that none of the five arms checks its context before `stuckOrCancelled`, so the new branch calls `cancelErr(ctx, <row>)` first in each arm.
  A new test: a transport push error under an already-cancelled context returns an error wrapping `context.Canceled`, and `shedtransient.Class` of it is empty.
- **F3** (Worktree-Teardown remote deletion): the out-of-scope list gains an entry for batten's Worktree-Teardown.
  `fabricengine.Remove` reports a failed remote branch deletion as text (`RemoveResult.RemoteBranchError`), so the error has no chain to classify.
  Widening that result is beyond the decision record's scope.

## Deferred

None.
