# lifecycle-waits: orch compaction, batten waits and reed strand windows

All three batches recorded `done`.

## Batch 01 — orch-compact-boundary (head `3034635da`)

Cards 1 to 4.
The orch compacting phase completes on a boundary found by `CompactedSince`.
A same-mode cycle request queued during a cycle is removed when the cycle's action takes effect.
`shedrun` declares the watched-marker path, and the driver stencil's notify rule is rendered from it.
`shuttleengine` gained a provider-neutral reading of a worktree's live runs' activity and API errors, with the Claude specifics in `claudeengine`.
Reported deviation: `cmd/lyx/testdata/test-budget.yaml`.
This batch's report was consumed on resume from a previous session's leftover.

## Batch 05 — run-shed-waits (head `e0ce9db62`)

Cards 5 to 8.
The Run-Shed row waits on the child in-call, state-changed notices are timed on the stop report with `since`, `history` and `report`, batten writes or removes the watched marker, and quiet is judged from agent activity with a new `api-error` notice.
The first fork hit its turn limit without a report, so the batch was re-forked once; the second fork committed card 8 on top of cards 5 to 7 already committed.
Reported deviations: `cmd/lyx/testdata/test-budget.yaml`, `internal/testkit/envkit/envkit.go`, `internal/loomcli/wiring_seams_test.go`, `internal/battencli/wire_test.go`, `internal/battencli/records_integration_test.go`, `internal/battenshed/innerrun_wait_test.go`, `internal/battenshed/seamchild.go`, `internal/battenshed/seamchild_test.go`.

## Batch 09 — reed-strand-window (head `ea8266c4d`)

Cards 9 to 11.
Reed addresses its strand window by id through one seam, a new `OpenWindow` op opens a detached named window running `lyx`, and `lyx batten run <slug> --window` plus the orch stencil's start step use it.
Reported deviations: `cmd/lyx/testdata/test-budget.yaml`, `internal/reedengine/overlay_test.go`, `internal/reedengine/spawn_test.go`, `pattern/PATTERN-pane-binary-resolution.md`.

The plan-level verify is left to the gate.
