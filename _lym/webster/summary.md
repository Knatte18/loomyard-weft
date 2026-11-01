# Configurable action buttons on the reed status bar

Both batches recorded `done`, with no warnings and no pattern departures.
The digests carried no renames.

## Batch 01-shell-detached-forms

Covered cards 1 to 5, head `a423899bdfe20bba4c38857502bc303fca232cf9`.
It added the detached pane-shell forms (output discard, zero exit, command flag, quoted-word directory change) and the `buttons` key in `reed.yaml` and `desktop.yaml`.
It also added button entry decoding, validation and filling, the lazy session place, placeholder and machine-button context from the hub geometry, and status line 3 with `follow` moved onto it and the global formats re-pinned on a stale build.

Scope-guard deviations (informational):
- `internal/hubgeom/hubgeom_test.go`
- `internal/reedengine/buttons.go`
- `internal/reedengine/buttons_test.go`
- `internal/reedengine/config_test.go`
- `internal/shell/shell.go`

## Batch 06-render-button-row

Covered cards 6 to 9, head `0bf24f56c235c103798442a4361c6467899cf142`.
It renders the buttons into the session's row and spec options at the status pins and sets the full strand name on each strand pane.
It also added `lym reed button`, which confirms, runs, logs and reports a clicked button, and bound a click on a button range to that verb in the run directory.

Scope-guard deviations (informational):
- `internal/reedengine/buttonclick.go`
- `internal/reedengine/buttonclick_test.go`
- `internal/reedengine/buttonrow.go`
- `internal/reedengine/buttonrow_test.go`
- `internal/reedengine/windowsize_test.go`
