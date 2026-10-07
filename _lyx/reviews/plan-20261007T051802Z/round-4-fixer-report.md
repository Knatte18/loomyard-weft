# Round 4 fixer report

All nine findings are fixed in the plan files; nothing is deferred.

## Changes

- **F1** (`04-drop-model-assertion.md`)
  - Added `acceptaudit_test.go` and `restoreplan_test.go` (both in `internal/webstercli`) to the Edit targets, and said their `initState` calls drop the argument.
- **F2** (`07-delete-blocked-by-later-card.md`)
  - The stencil edit now searches the whole stencil for every line that sends a failed batch to `recover-batch` and adds the plan-edit exception to each.
  - It keeps the claims-pinned clauses verbatim.
- **F3** (`07-delete-blocked-by-later-card.md`)
  - The recover-batch refusal wraps a new sentinel, `ErrRecoveryDeleteReferenced`, which `internal/webstercli/recoverbatch.go` maps to the existing `batch_failed` flag.
  - That file is now an Edit target, and the cobra test asserts the flag.
- **F4** (`08-background-shell-outcome.md`)
  - Added `internal/websterengine/doc.go` to the Edit targets, with a Docs paragraph rewriting the background-shell section.
- **F5** (`11-reset-moves-remote-branch.md`)
  - The card now states that `SkipGit` does not skip the fetch or the remote update, and gives the reason.
- **F6** (`04-drop-model-assertion.md`)
  - Only W2a and W2b and the `assertedModel` tamper exception are removed from the sandbox suite.
  - W2c moves into W1.
  - The `modelswitch_test.go` comment citing W2b is reworded, and the file is an Edit target.
  - The ImpactSummary matches.
- **F7** (`00-overview.md`)
  - The `cli-drive` rationale now gives the true reason for driving Publish below the CLI: no loomcli test steps Publish, so a CLI drive needs a new harness.
  - It says what verifies the D8 item instead.
- **F8** (`11-reset-moves-remote-branch.md`)
  - Named the stale tallies in `destructiveguard_test.go` and `internal/fabricengine/doc.go` and said each is reworded to count nothing.
- **F9** (`10-fabric-remote-only-commits.md`)
  - `CommitsNotIn` is called with the resolved HEAD SHA.

## Deferred

None.
