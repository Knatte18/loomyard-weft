# Fixer-report, round 1

## Changes

- F1 (fixed as proposed, commit `8e17d911`): `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` in `internal/battencli/wire_seams_test.go` now sets `wantState, wantCause` to `idle-stalled`, `no-output` unconditionally.
  The `liveness-unproven` default, the `proc.StartTime` registry-entry setup with its comment and the unused `proc` import are deleted.
- F2 (fixed as proposed, commit `9407c4e5`): the `--anchor` guard in `internal/shuttlecli/run.go` returns `invalid --anchor %q; want own-window`, and appends the reed explanation only when the value is `hidden`.
  In `internal/shuttlecli/cli_test.go` the `AnchorHidden` row asserts the full hidden text and the `AnchorUnknown` row asserts the plain text.

`lym gate test ./internal/battencli ./internal/shuttlecli` passed with 0 failures.

## Disputed

None.

## Deferred

None.

## Self-check

All four forks returned a report; none returned "no report".

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
