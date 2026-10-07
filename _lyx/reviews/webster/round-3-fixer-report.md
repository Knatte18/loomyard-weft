# Round 3 fixer report

All three findings are fixed, each in its own commit, none pushed.
After the last commit `go build ./...`, `go vet ./...` and `go test ./...` pass.

## What changed

- **F1** (`82c0b17f2`): `lyx loom resume` now creates the directory of the status lock before reading the status file, and the directory of the run lock before probing it, in `internal/loomcli/resume.go`.
  The existing `TestResumeVerb` table gained one row, "running with the scratch directory absent answers from the run's state", which points the lock paths at a directory that does not exist after the status file is written.
  The row failed before the fix (ok = false, exit 1) and passes after it.
- **F2** (`c6d054a19`): broke the `TestShuttleWaiting` comment in `internal/loomcli/reviewstatus_test.go` after each of its three independent clauses.
- **F3** (`1340c2e5d`): broke the changed markdown lines the finding names at their independent-clause boundaries, keeping the wording:
  `pattern/PATTERN-completion-signal.md` (two lines), `contracts/specs/loom-status-spec.md` (two lines), `contracts/stencils/orch/orch-template-role.md` (one line) and `docs/overview.md` (the halted-child bullet and the dead-driver bullet).
  The adoption bullet in `docs/overview.md` had the same shape, ", and the bootstrap itself is idempotent …", and was broken too.
  `go test ./contracts/...` passes, so no stencil claim split across the new line breaks.

## Deferred

Nothing.
