# Hide batten run windows from reed's status bar

`lyx batten run --window` keeps starting each run in its own detached window of the prime's reed session, since the batten process lives there.
Those windows no longer get a `batten:<slug>` button on status line 1, because their output tells the operator nothing useful.
Runs are watched through `lyx batten status` and each run's own terminal window until the runs overview exists.

## Design

- `OpenWindow` in `internal/reedengine/window.go` sets the window user option `@lyx_hidden` to `1` in the same chained tmux invocation as `remain-on-exit off`, addressed by exact window name.
  Setting it in that invocation means the window is never visible without the marker.
- `statusFormatButtons` in `internal/reedengine/bar.go` now draws the fallback per-window button only for a window with neither `@lyx_strands` nor `@lyx_hidden`.
  The existing multi-window condition is kept.
- `@lyx_hidden` is display only; no Go code reads it.
- Strand buttons are unaffected, since they key off `@lyx_strands`.

## Docs

- `internal/reedengine/doc.go` describes the marker and the changed purpose of detached windows.
- The orch role stencil (`contracts/stencils/orch/orch-template-role.md`) no longer says the window list names every run in flight, and points at `lyx batten status <slug>`.

## Tests

The window and bar unit and integration tests assert the new chained option and that a hidden window gets no button.
The integration tier needs a real tmux; it was not run here.
