# shuttle-send: sends that land, and stops that are recorded

All three batches reported `done`.

## Batch 01 — shuttle-send-config (head `8f5febbf7`)

Cards 1–6 landed as one commit each.
Shuttle config gained `send_ready_timeout_s` and `submit_confirm_timeout_s`, and the two settle keys were reworded.
The provider seam gained typing without submit, a clear choreography and a paste-placeholder reading.
Every verified send now waits for an idle session or fails `ErrSessionBusy`.
A send settles before Enter, confirms within a window, and on `ErrSubmissionNotLanded` clears its own text.
A start-time send or persist that fails tears the run down as not started.
Gate sends wait for an idle writer, and an unlanded re-prompt retries without spending an attempt.

Deviations: `cmd/lyx/testdata/test-budget.yaml`, and `internal/shuttlecli/cli_test.go`, where an existing test now asserts the died outcome.

## Batch 07 — shuttle-stop-verb (head `d92c66e6c`)

Cards 7 and 8 landed.
Shuttle's stop verb records `stopped` before it removes a run's strand, attach uses it, and burler stops a half through its run handle's `Stop`.

Deviations: `cmd/lyx/testdata/test-budget.yaml`, and `internal/shuttleengine/run.go`, where the prompt-offset save goes through the new record lock.

## Batch 09 — webster-stops-through-shuttle (head `c91f203d7`)

Cards 9 and 10 landed.
Webster's reclaim stops through `Runner.StopStrand`, `PATTERN-shuttle-stop` was added, and burler's, shedfake's and webster's strand removers were deleted.

Deviations: `cmd/lyx/testdata/test-budget.yaml`, `internal/websterengine/stopper_test.go` and `internal/websterengine/recoverbatch_test.go`.
