# runs-overview: render sizing, shared status pane, runs strand and run-window close

Both batches reported `done`.

## Batch 01 — render-size-vocabulary (cards 1–6)

Head `4a31fd0f0`.
Built render's top anchor, content and sharing size kinds with `ValidateDisplay`; reed's content-row persistence, title-row planning and top-strand splits; `Engine.SetContentRows`; the display-only Selvage marker in the Alt+Up/Down cycle with Alt+z; the new `internal/statuspane` package (pane loop, frame fitting, content-rows report, status cache); and the move of loom's build identity and sidecar into `statuspane`.
Deviations: `internal/reedengine/attach_test.go` and `internal/reedengine/watchdog_integration_test.go`, updated for the changed `planLayout` signature.
The `tmux`-tier edits were compile-checked only.

## Batch 07 — loom-checklist (cards 7–12)

Head `912644437`.
Built the hidden loom checklist verb in the loom-status strand, the once-per-process batten notice log in the hub, the hidden batten overview verb, the orch runs strand above the orch, sharing kinds on seat-table strands, and the run window closing on a 0 attach or a removed pair.
Deviations: `internal/loomrecipe/seam_enforcement_test.go` (import allowlist), `cmd/lyx/hermeticenv_test.go` (allowlist), `internal/battenshed/rownotice.go`, `deps.go` and `notice.go`, `internal/shedrun/paths.go`, `internal/orchcli/start.go`, `internal/seatengine/prompt_test.go`.
The new `tmux`-tier reedengine test was compile-checked only.
The fork reported two places where the code departs from card wording: a lone top status strand is pinned at the full box height, not its content height, and the overview's hidden-runs summary counts every run not shown ("… 2 more runs not shown").


## Integration suite triage

The plan-level `## verify:` suite failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed`
