# bugfix-webster: plan-order gates and Webster, reset and Publish stops

All twelve batches recorded `done`, each committed in its own commit, with no recovery needed.

## Plan-Gate

- **01 `c9e8bf123`:** `planparser` refuses a `Uses:` that names a later card's target (`uses-later-target`).
  The regression runs through `lyx webster validate`, and the existing handle fixture was reworked to stay valid.
- **02 `f451782d1`:** `planglyph.LaterDeleteReferences` refuses a delete that a later card's Edit code still references (`delete-before-reference`).
  Deviation: `internal/planglyph/status_enforcement_test.go` gained the new fail-closed status reader in its allowlist.
- **03 `d3b1b4660`:** `ValidateDispatch` downgrades an already-deleted Delete target of an unbegun card to an informational `delete-target-gone`, once at least one card is begun.
  The whole-plan entry points still refuse it.

## Webster

- **04 `90f1aa609`:** `begin-batch` no longer asserts or injects Master's model; the injector seam, `AssertedModel` and the matching sandbox assertions and spec row are gone.
  Deviations: `internal/webstercli/cli_test.go` and `internal/websterengine/beginbatch_git_test.go`, both needed to compile.
- **05 `2f86cd5ce`:** `begin-batch` archives a report that has no begin-batch record and returns `archived_report`; a recorded batch's report still refuses, with a clearer reason.
- **06 `4620a0791`:** `rebaseline --card` accepts an edit to a failed, dead or stuck batch's card and restamps only that card's hash.
- **07 `de976e947`:** a `delete-not-done` caused by a later card names it with a plan-edit way forward, and `recover-batch` refuses with `batch_failed` instead of looping.
- **08 `5821c3427`:** the background-shell note and warning share one sentence stating what happened and the run's outcome.

## Reset and Publish

- **09 `9905e3b6c`:** `gitrepo` gained `UpdateRemoteBranchLeased` with `ErrLeaseRejected`, and `CommitsNotIn`.
- **10 `9416174c4`:** `Fabric.RemoteOnlyCommits` returns the remote task branch's tip and the commits HEAD lacks.
- **11 `aeecd60be`:** `webster reset` moves the remote task branch under an ancestry gate and a lease, and a failure after the remote moved reports `partial` with the mutation record.
- **12 `f07c432cd`:** Publish's push-rejected halt names the remote tip, the count of commits HEAD lacks and the resume.
  Deviations: `internal/testkit/envkit/envkit.go`, `internal/shedrecipe/entries_simple_test.go`, `internal/loomcli/wiring_seams_test.go` and `internal/battencli/wire_seams_test.go`, all needed for the new `Deps.RemoteOnlyCommits` field.

## Verify gate

The plan-level `## verify:` was not run by Master; the gate runs it at HEAD.
