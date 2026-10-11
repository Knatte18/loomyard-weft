# pane-inject-safety: guarded pane writes, paste path and driver_busy

All three batches (cards 1–17) recorded `done`; none needed recovery.
The plan-level verify runs at the gate after this turn.

## Batch 01 (cards 1–6), head `d2aca5854`

Reed gained a one-invocation paste-submit op and an operator-activity query with the `operator_quiet_s` key.
The watchdog's `/rename` now sits behind the activity check and the paste-submit.
Shuttle's `ReedOps` seam and both fakes carry the two new reed ops.
Shuttle holds a write for an active operator and for an unreadable input box, pastes through reed for a `TextPaster` engine, and sends its own Enter only on an exact match.

Deviations from the declared file-ops: `internal/shuttleengine/engine.go` (the `TextPaster` interface), `run.go`, `fakes_test.go`, `run_test.go` and `startup_test.go`.
The fork reported that the floor rows for `operatorQuiet` sit in `io_test.go` because `config_test.go` is an external test package.
It also reported that the delivery-verification row went into `TestRun_Send_ConfirmsSubmission` instead of `TestRun_Send_DeliveryVerification`.
The new `tmux`-tier tests were vetted with `-tags tmux` and not run.

## Batch 07 (cards 7–11), head `faeedf662`

Shuttle records a send that may have left text in the box and dedupes its retry.
`/clear`, `/compact`, `/reload-plugins` and `/color` go through the guarded path.
Claude moved onto the paste path without the leading Escape, with the PATTERN entry added.
An `llm`-tier smoke test pins Claude Code's bracketed-paste behaviour; it was compiled with `-tags llm` and not run.
The orch watcher waits out a held write instead of failing the tick.

Deviations: `internal/shuttleengine/config.go`, `session_test.go`, `claudeengine/startup_test.go` and `internal/shuttlecli/modelpin_test.go`, where `wantSpawnSiteCount` was raised to 13 for the new real-`claude` spawn site.

## Batch 12 (cards 12–17), head `965081117`

`shedrun` declares the `driver_busy` kind and the resume report file.
Loom refuses a busy driver pane with `driver_busy` and reuses one report path per park.
Batten retries a busy driver pane and notices the orch once per stall, and maps the child's refusal to its sentinel.
Batten status flags a halted batten whose child loom run is done, and the orch's after-landing routine closes batten.
The first batch-gate run failed in `TestBattenIntegration_Rows` because the card 16 step ran after the dirty-prime step; the fork moved it ahead in its own commit and re-ran `battencli` alone green.

Deviations: `cmd/lyx/testdata/test-budget.yaml` (the `loomcli` row 195 to 196, the `battencli` row 58 to 59), `internal/battenshed/notice.go` and `deps.go`, `internal/loomcli/start.go` and `resume.go`, and `internal/battencli/lifecycle_integration_test.go`.
