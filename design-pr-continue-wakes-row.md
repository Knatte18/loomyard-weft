# pr-continue wakes the PR-Review wait at once

`pr-continue-wakes-row` · Note · enhancement, landing, batten

## Seen (operator and hub orch, 2026-10-30)

On the first PR-Review run, the reviewer agent ran `lym loom pr-continue` and the run went on only after a while.
The wait sleeps between wakes, from a floor of one second up to a ceiling of one minute while nothing changes (`prWaitFloor`, `prWaitCeiling` in `internal/landingshed/prreview.go`), and `pr-continue` only sets a flag that the next wake reads.
Finalize's gate then runs on the landed tree as for any landing, which takes minutes by itself.

## Design (operator, 2026-10-30)

`lym loom pr-continue` wakes the waiting row at once, so the PR is closed and Finalize begins with no wait of up to a minute.
