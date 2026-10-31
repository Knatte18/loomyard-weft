# batten: resume a Worktree-Teardown whose pair is already gone

A batten run blocked at Worktree-Teardown after its pair was removed could not be resumed.
`lym batten run <slug>` ran the run-scope origin reconcile first, and that reconcile refuses a slug with no pair worktree.
So a teardown whose only leftover was the remote branch deletion, such as after a GitHub 500 on `git push --delete`, named a way forward that could never work.
`lym batten cancel` refused the landed run and pointed at the same verb, so no verb could end the run.

Now a resume whose status stands at Worktree-Teardown and whose task worktree is absent skips the reconcile, and the teardown retries what is left.
The failed-deletion error names `lym batten run <slug>` as the retry.

## Design decisions

- The skip covers only that producer with that worktree absent.
  Every other resume, including Worktree-Teardown with the pair present, still runs the reconcile.
  The decision sits in `ownerPreflight` in `internal/battencli/owner.go`.
- With the reconcile skipped, the owner is judged on the local board alone.
  A run the board shows owned by another machine is still refused, naming that owner and `lym adopt <subject>`.
  `refuseResumeUnreachable` became `refuseOwnedElsewhereOnLocalBoard(retry)`, so the "re-run once origin answers" hint stays with the unreachable-origin path.
  A re-run would skip the reconcile again, so the teardown refusal omits it.
- The skipped reconcile writes no origin-owed marker.
  The marker asks a later pass to reach origin for a reconcile this run can never run again, and the teardown's own pushes already go to origin.
- `internal/pairteardown`, `lym batten cancel`, `lym batten cleanup` and `fabricengine.ReconcileOrigin` are unchanged.
- The failed-deletion error in `finishRemoval` (`internal/battencli/wire.go`) now reads "re-run lym batten run <slug> to retry the deletion".
- Docs follow the behavior: the `internal/battencli` and `internal/battenshed` package docs, the resume comment in `arm.go`, `docs/overview.md`, and `pattern/PATTERN-owner-machine.md`.

## Tests

- `TestOwnerPreflight_TeardownResume` (tier0, in `owner_test.go`) pins the skipped reconcile with the local board owned here, unowned and owned elsewhere, the reconcile still running with the pair present and at another row, and no origin-owed marker in any case.
- `TestRunCmd_ResumeDispositions` and `TestBattenPreStep_RefusalKinds` moved their reconcile-running rows from Worktree-Teardown to Land, since a Worktree-Teardown resume with no pair no longer reconciles.
- The wire and lifecycle-integration tests assert the new error wording.

## To check by hand

- A run whose task worktree vanished by hand before Worktree-Teardown ever ran now resumes without an origin judgement; the teardown removes what is left, the same outcome `lym fabric remove` gives such a pair.
- Resuming a real run blocked at Worktree-Teardown with its pair removed and a remote branch left undeleted was not run against a live remote.
- Review findings left open, to confirm they are settled in the diff:
  - `internal/battenshed/doc.go`: its resume pre-flight description named only the reconcile paths, and now carries the Worktree-Teardown exception.
  - `pattern/PATTERN-owner-machine.md`: its origin-reconcile-first and origin-owed-marker rules did not name the teardown exception.
  - `internal/battencli/owner_test.go`: the Worktree-Teardown resume with the pair present, which must still reconcile, is covered by the `TeardownWithPairPresentRunsTheReconcile` row.
