# Reed down ends its watchdog; strand panes resolve lyx to the spawning binary

Both batches recorded `done`.

## Batch 01-lyx-bin-env-key

Covers cards 1–3: the `LYX_BIN` key declared in `agentname` and read by reed's prelude, the Claude engine's env file re-prepending `LYX_BIN`'s directory, and `Engine.Down` writing the discover stop marker.
Head `f1404e01d5f30e13a631eaf89d7471a74d60b6a1`.
Deviations the scope guard listed: `internal/shuttleengine/claudeengine/envfile_integration_test.go`.

## Batch 04-watchdog-stop-and-recheck

Covers cards 4–6: the watchdog daemon exiting on a stop and re-checking on exit, the down verb waiting, bounded, for the watchdog lock, and the sandbox suites naming the shell their lyx check runs in.
Head `fc6fa0af64289516fd0cde36251b4cba5d06ad59`.
Deviations the scope guard listed: `cmd/lyx/testdata/test-budget.yaml`, `internal/reedcli/cli.go`, `internal/reedcli/watchdog_test.go`.

## Scope guard

Each batch's deviations are informational only; neither batch was stuck or recovered, and no record-batch warnings came back.
