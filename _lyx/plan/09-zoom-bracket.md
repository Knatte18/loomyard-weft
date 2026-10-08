# Card 9 — A zoomed strand survives reed's own steps

**Edit:**
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/strand.go`
- `internal/reedengine/replace.go`
- `internal/reedengine/reapply.go`
- `internal/reedengine/attach.go`
- `internal/reedengine/windowsize.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedengine/reapply_test.go`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine/zoom.go`
- `internal/reedengine/zoom_test.go`
- `internal/reedengine/zoom_integration_test.go`

**Intent:**
Make the zoomed single-strand view of D1 survive every reed step and every resize, with its bound: a step re-zooms only a window that was zoomed when it started, and only the pane that was zoomed; a pane gone by then leaves the window in the overview; layout and pins always run on the unzoomed window, so the overview's cells are correct at the next unzoom.
tmux clears `window_zoomed_flag` on `resize-pane -y`, `select-layout`, `split-window`, `select-pane` without `-Z` and `kill-pane` of another pane, which is why the bracket wraps every pane-changing step rather than the two D1 first named.

Go bracket (`zoom.go`): `(e *Engine) keepZoomLocked(fn func() error) error`, called under the op lock at the start of each step.
It resolves the strand window through the stored-state seam (`storedStrandWindowTarget`), reads `display-message -p -t <window> '#{window_zoomed_flag} #{pane_id}'` (the active pane is the zoomed one), and, when zoomed, unzooms with `resize-pane -Z -t <pane>`, runs fn, and afterwards, also when fn returned an error or a guard ended it early, re-zooms with `resize-pane -Z -t <pane>` only if that pane is still in the window's listing.
No session, a failed read or an unparseable answer means "not zoomed": fn runs plain, logged at debug.
Every bracket call is non-fatal and logged.

Wrapped steps, each inside its own `withOpLock` closure: `Up`, `EnsureSession`, `Resume`, `AddStrandUnless` (and through it `AddStrand`), `UpdateStrand`, `RemoveStrand`, `ReplaceStrand`, the watchdog's `reapplyLayout`, and `AttachArgv`'s pre-flight.
So every pane list a layout or a pin is planned from is read on the unzoomed window: a zoomed pane reports `pane_top` 0 and the window's full height, which would map planned heights onto the wrong panes.

Run-time bracket for the two executions tmux runs after reed returns, built as tmux commands that test the zoom when they run, not when reed builds them:

- `resizePinHookArgvs` (`windowsize.go`), when it emits any pin, emits first an entry `if-shell -F '#{window_zoomed_flag}' 'set-option -w @lyx_rezoom 1 ; resize-pane -Z'` and, after the pins and before the watchdog signal entry (which stays last), an entry `if-shell -F '#{@lyx_rezoom}' 'resize-pane -Z ; set-option -wu @lyx_rezoom'`, re-zooming the active pane, which is the one that was zoomed.
- The attach chain (`attach.go`) runs the same record-and-unzoom, then its `select-layout`, then the pins of card 8, then the same re-zoom, each with an explicit `-t <strand window>` since the chain runs in the client's context.
  Its layout string is planned under the op lock from a pane listing taken inside the Go bracket (unzoom, list, re-zoom), so the planned order ignores zoom.

Tests:
`zoom_test.go` (fake tmux) is one table over `keepZoomLocked`: not zoomed (no `resize-pane -Z`), zoomed (unzoom, fn, re-zoom of that pane), zoomed with fn erroring (still re-zoomed), zoomed pane gone after fn (no re-zoom), no session (fn runs plain).
`windowsize_test.go` asserts the hook array's entry order: record, pins, re-zoom, signal; and no bracket entries with zero pins.
`attach_test.go` asserts the pre-flight's unzoom-list-re-zoom order and the chain's order; `reapply_test.go` asserts the watchdog re-apply is bracketed.
`zoom_integration_test.go` (`//go:build tmux && linux`, the pty attach harness) zooms a strand that is not the top pane and asserts it is still the zoomed pane after a window resize, a re-attach, an add and a resume; after the next unzoom every cell has its planned height; and a zoomed strand whose pane is removed leaves the window in the overview.

`doc.go` gains a zoom section: the bracket, its call sites, the run-time entries and the bound.

**ImpactSummary:** every pane-changing reed op and the attach pre-flight gain a zoom read and, when zoomed, two `resize-pane -Z` calls; the hook array gains two entries.
