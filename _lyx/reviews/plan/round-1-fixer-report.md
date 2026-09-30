# Plan-Review round 1 — fixer report

All six findings are fixed.
`lyx loom validate-plan` passes on the edited plan (`ok: true`, informational `create-new-unit` notes only).
`approved:` in `00-overview.md` is untouched.

## Changes

- **F1 (MEDIUM), `06-finalize-verify-gate.md`:** the retry scenario now scripts both merge-ins as tree-changed, with the first verify passing and the second failing, and the merger refusing its first `Merge` with `ErrMergeInRequired`.
  The assertions are: runner called exactly twice, Stuck, `Merge` called exactly once, `PushBranch` and `MarkTaskDone` never called, marker remains.
- **F2 (LOW), `05-publish-verify-gate.md`:** the failing-verify case now asserts the output file holds the line the fake runner wrote.
  A new tree-changed marker-write-failure case asserts a returned error, not Stuck, no push, and the runner never called.
- **F3 (LOW), `01-verifyrun-runner.md` and `02-webster-shared-runner.md`:** `verifyrun.Run` leaves `cmd.Stdout`/`cmd.Stderr` nil when `out` is `io.Discard`, so the child gets the null device as it does today and no copy pipe exists.
  Card 2 states this, and card 1's integration test runs `exit 3` with both a buffer and `io.Discard`.
- **F4 (LOW), `01-verifyrun-runner.md`:** the conditional `testmain_test.go` instruction is replaced by a statement that the package needs no `TestMain`, and why.
- **F5 (NIT), `03-loomengine-verify-paths.md`:** dropped `internal/loomengine#loomApprovalFileName` from `Uses`.
- **F6 (NIT), `01-verifyrun-runner.md`:** the card also adds `internal/verifyrun` to `docs/overview.md`'s shared-infrastructure package list, after `internal/shell`.

## Deferred with reason

None.
