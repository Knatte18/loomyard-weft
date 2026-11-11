# No change: the died-driver smoke case passes on main since 380101d7

The board entry reports that `internal/loomcli` `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` fails at `smoke_test.go:738`.
In that run, the driver log carried `shedverbs: step interrupted (loop-exited)` where the test waits for the poisoned status file's `invalid state`.
That failure was seen on base `dd1be105`.

This change leaves the code and the test unchanged.
On base `2136b28b`, `lym gate test --tier 2 ./internal/loomcli -- -run TestSmokeBootstrapLifecycle -count=1` passes all six subtests, the died-driver and malformed-status cases among them.

## Why it passes now

Main's `380101d7` added the stop's `detail` field to the driver's `sheddriver: drive: handing the run back` log line in `internal/sheddriver/drive.go`.
The failing log line on `dd1be105` carried only the error text of the loop-exited stop, with no detail.
From `380101d7` on, the line also carries the detail of the loop-exited stop, and the driver log names the halt reason again.
The case fails only on bases before `380101d7`.

## Design decisions

- No code or test change.
  The run's parent decided to settle the entry with this report once the test passed on the current base.
- The driver log text of a passing run was not captured, so this report does not quote the exact clause that carries `invalid state`.
  The parent declined a second, diagnostic run.

## To check by hand

- Which clause of the loop-exited detail carries `invalid state` in the driver log: the loop log excerpt or another clause.
  The comment above the test's assertion still calls the text "the read gate's invalid-state refusal" and does not say it now reaches the log through the stop's detail.
