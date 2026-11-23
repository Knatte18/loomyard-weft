# Name the run in a darn verify-budget notice

A darn run's verify-budget notice to its parent orch now opens with `loom <slug>: `.
Before, the line read "Darn-Write gate: the verify gate has run past its 30m0s budget and is at attempt 3 ...",
and neither the sender (`ly:LOOM`) nor the text named the run.
The orch reads the notices of every run of the hub, so with several darn runs in Darn-Write it could only guess which run a notice was about.

## Design decisions

- The prefix is added at the notify seam in `treeScopeSettings` (`internal/loomcli/darnwiring.go`), not in `verifyBudgetNotice` (`internal/loomshed/verifybudget.go`).
  `loomshed` knows the gate site but no slug, and the seam already holds the run's `location`.
  The slug is `location.WorktreeName`.
- The `loom <slug>:` form follows the other parent notices.
- `internal/loomshed` is unchanged, so the line's own wording and the budget logic stay as they were.

## Tests

`TestVerifyBudgetNotice_NamesTheRun` in `internal/loomcli/notifystamp_integration_test.go` drives `treeScopeSettings` on a pair worktree of a real hub.
It posts a notice through the returned `Notify` and checks the subject creel received opens with `loom budget-task: `.
It is a tier 1 integration test and was not run as part of this description.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
