# Resume a Worktree-Teardown whose pair is gone with no origin reconcile

A batten run whose Worktree-Teardown removed the pair but failed to delete the other-side branch on origin stayed `blocked`, and could be neither retried nor cancelled.
`lym batten run <slug>` refused before the retry, because its resume pre-flight runs the run-scope origin reconcile and that reconcile refuses a slug with no pair worktree.
`lym batten cancel` refuses a run whose work reached its parent and names `lym batten run` as the way forward, so no verb could end the run.

## What changes

- `ownerPreflight` in `internal/battencli/owner.go` skips the origin reconcile when the status file stands at Worktree-Teardown and the slug's task worktree is gone.
  The owner is then judged on the local board alone, as for an unreachable origin, and the resume runs the teardown row, which already finishes a half-removed pair, deletes what is left on origin and reports a pair of which nothing remains as done.
  Every other resume, a teardown whose task worktree is still there included, runs the reconcile as before.
- The local-board owner refusal (`refuseOwnedElsewhereOnLocalBoard`, formerly `refuseResumeUnreachable`) takes the retry clause from its caller.
  An unreachable origin keeps "re-run `lym batten run <slug>` once origin answers"; the teardown case names only `lym adopt <subject>`, since a re-run would skip the reconcile again.
- The Worktree-Teardown row's failed-deletion error names `lym batten run <slug>` as the retry instead of "resume this run".
- `lym batten cancel` is unchanged: its way forward for a landed run now works.

## Design decisions

- The skip writes no origin-owed marker: the marker asks a later pass to reach origin for a reconcile this run can never run again once its pair is gone.
- The skip is bounded to the Worktree-Teardown row with the task worktree absent. What it lets through is a resume whose run origin has ended or stopped, judged only locally; that resume can only run the teardown, which removes leftovers of the pair and its remote branches. An entry the local board shows owned by another machine is still refused.
- `fabricengine.ReconcileOrigin`'s run scope keeps refusing a slug with no pair worktree; the fix stays in the one resume caller that expects a gone pair.

## Tests

- `TestOwnerPreflight_TeardownWithPairGone` pins the skip: no reconcile and no marker, a resume when owned here, an owned-elsewhere refusal naming the adopt without "once origin answers", and the reconcile still running at another row.
- The resume-disposition and pre-step fixtures that used Worktree-Teardown with no pair on disk as a generic resumable row now use the Land row, so they keep exercising the reconcile.
- `TestFinishRemoval` asserts the new retry wording.

## Docs

The battencli package doc, the resume comment in `internal/battencli/arm.go` and the Worktree-Teardown paragraph of `docs/overview.md` state the skip.
