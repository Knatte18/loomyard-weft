# ubuntu-2404-support: stuck at batch 01

No batch finished, so the run ends `stuck` and the later batches (03, 04, 06, 09) were never begun.

## Batch 01-lyxtemp-short-run-names

The batch holds cards 1 (`lyxtemp-short-run-names`) and 2 (`toolfloor-package`).
The fork committed card 1 (`c27d4e829`), which names run directories `r` plus six hex digits.
It halted card 2 on the audit rule: the fork reported that tmux's `allow-set-title` needs tmux 3.5, above the 3.4 floor, while `internal/reedengine/spawn.go` only warns when it is missing.
It wrote no card 2 code, so `record-batch` refused with `batch_failed`: the `internal/toolfloor` targets do not resolve.
`recover-batch 01` refused with `batch_failed` again, for the same missing targets.

## Way forward

The plan needs a decision on `allow-set-title`: either a third recorded form (found above the floor, tolerated by the Warn fallback) or a replacement.
The run resumes with `lyx webster run` once the plan is edited.

Digest fields: no `deviations`, `departures` or `dead_reason` were read, since neither verb returned a done digest.
