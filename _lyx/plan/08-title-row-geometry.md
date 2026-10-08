# Card 8 — Geometry under the title row: adjusted pins run at every apply

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/reedengine/windowsize.go`
- `internal/reedengine/apply.go`
- `internal/reedengine/attach.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/apply_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedengine/attachgeometry_integration_test.go`
- `internal/reedengine/doc.go`

**Intent:**
Keep every planned cell at its planned height with `pane-border-status top` on (card 5) and the two-row status line, per D7.
The layout planner and `render` are unchanged: the row budget already takes the reserved rows from the `#{status}` readback, and the layout string still sums to the window height, since tmux draws the title row inside the top cell.

Readback: `windowsize.go` gains `readBorderTitleRowLocked(windowTarget) bool`, `display-message -p -t <window> '#{pane-border-status}'`: only an answer of exactly `top` means a title row at window row 0; `off`, `bottom`, an empty answer and a failed readback (logged) all mean none.

Adjustment: a pure `pinsForContent(pins []render.Pin, rowZeroPaneID string, titleRow bool) []render.Pin`.
A `resize-pane -y` pin sizes a pane's content, which equals its cell everywhere except at window row 0; so with a title row the pin on the row-0 pane is its planned cell height minus one, floored at one content row, and every other pin, Selvage's included, keeps its cell height.
The floor is the one case where a cell ends one row taller than planned, the row coming from the cells below it.
The row-0 pane is the first entry of the physical pane order the applied `select-layout` string was built from (`toRenderInputs`'s `paneOrder`), never the pin emission order, since a pin carries only a pane id; a row-0 pane with no pin gets none.
Without a title row the pins pass through unchanged.

Runs:

- `applyLayoutLockedOpts` (`apply.go`), right after its `select-layout` and on both paths, the focusing one and the watchdog's `SkipFocus` re-apply, issues the adjusted pins itself as `resize-pane -t <pane> -y <n>` calls, so the floor holds from the apply, not only after the next resize.
  This run issues pin entries only, never the hook's watchdog signal entry, so an apply never signals the watchdog into another apply.
  Each call is non-fatal and logged, like the hook install (Shared Decision hook-failure-is-non-fatal-everywhere).
  The focusing path still installs the `window-resized` array, now with the adjusted pins.
- `AttachArgv` computes the same adjusted pins against its told box and its own pane listing, installs them in the hook as today, and appends them to the chain after its `select-layout`, one `;`-separated `resize-pane -t <pane> -y <n>` per pin (`chainedAttachArgv` takes the pins).

A Selvage-only window has no pin and changes nothing.

Tests:
`windowsize_test.go` is one table over `pinsForContent` (title row with a multi-row top cell, a one-row top cell floored, the row-0 pane absent from the pins, no title row) and asserts the hook array carries the adjusted heights; a fake-tmux row over the readback maps `top`, `bottom`, `off`, empty and an error.
`apply_test.go` asserts both apply paths issue the pins after `select-layout` and that the `SkipFocus` path, the one the watchdog re-apply takes, issues no `set-hook` and no signal entry; `attach_test.go` asserts the chain's argv order (`attach-session ; select-layout … ; resize-pane …`).
`attachgeometry_integration_test.go` (`tmux && linux`): the existing chain-shape checks stop assuming the 10-element chained form and taking the layout string from `argv[len(argv)-1]`; they assert the chained form by the presence of its `select-layout` element and take the planned layout string from the argument after it, and the 5-element bare-form checks stay.
Its collapsed-parent height checks keep expecting `cfg.CollapsedRows` minus the title row (card 5), which this card's adjusted row-0 pin now holds after a resize as well as at attach.
The file gains the D7 acceptance rows with the two-line status and `pane-border-status top`: the applied layout fills the window exactly with no rescale or overflow at boot and after a resize; after a resize with the hook installed every cell, the top one included, has its planned height, also with the strand panes in a physical order other than the intended one; with `pane-border-status` set to `off` and to `bottom`, every pin keeps its cell height and the cells keep their planned heights; a planned one-row top cell ends one row taller with one content row right after the apply, before any resize.
`test-budget.yaml`'s `internal/reedengine` row is set to the package's top-level test count after this card.

`doc.go`: the resize round-robin bullet and the chained-attach bullet describe the title-row adjustment, the pins run at apply, and the chain's pin tail.

**ImpactSummary:** every layout apply and attach chain gains a pin run; the hook array's top pin shrinks by one row when titles are on; planner and render untouched.
