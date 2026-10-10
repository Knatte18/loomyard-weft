# Webster audit gaps closed

Both batches recorded `done` on the first attempt, with no recovery.

## Batch 01: pathless-entries-never-resolve (cards 1–4)

Head `4b1d3748d177f55e871651194f371914d08db615`, no deviations.
Cards 1–4 landed as one commit each.
The read-only classifier in `internal/fabricengine/refscanner.go` was reworked, and `internal/websterengine/recordbatch.go` gained the pathless-entry handling.
Card 4's walk of the real command tree reports `batten status`, `loom status` and `shed status` as the commands declaring their own `--json`, and pins that set.
That raised the `cmd/lyx` test budget from 72 to 73.
`record-batch` returned only informational scope warnings for symbols in those files.

## Batch 05: git-reader-flag-rows (cards 5–8)

Head `1e332015aa0476459300021c77e37097198e47bb`.
One deviation: `internal/websterengine/merriambase.go`, where the `MerriamBase` doc comment was reworded.
Cards 5–8 landed as one commit each: git reader flag rows, the one-way-forward `MerriamBase` error, the `begin-batch` existing-report remedy with `recovery_exhausted`, and the thin auto-rebaseline and ended-shell rows.
The fork reported that it committed card 6 once with a failing test, then fixed the test and amended the commit; the amended commit passed its gate.
`record-batch` returned only informational scope warnings for the touched test tables and `MerriamBase`.
