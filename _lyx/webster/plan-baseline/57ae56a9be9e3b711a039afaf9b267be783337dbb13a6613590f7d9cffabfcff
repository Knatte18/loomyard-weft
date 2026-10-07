# Card 9 — reed-strand-window

**Create:**
- `internal/reedengine/windowtarget.go`
- `internal/reedengine/windowtarget_test.go`

**Edit:**
- `internal/reedengine/overlay.go`
- `internal/reedengine/windowsize.go`
- `internal/reedengine/selvagepane.go`
- `internal/reedengine/apply.go`
- `internal/reedengine/generation.go`
- `internal/reedengine/attach.go`
- `internal/reedengine/directory.go`
- `internal/reedengine/reapply.go`
- `internal/reedengine/io.go`
- `internal/reedengine/namerepair.go`
- `internal/reedengine/spawn.go`
- `internal/reedengine/strand.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/doc.go`
- `internal/reedengine/faketmux_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/attachgeometry_integration_test.go`
- `internal/reedengine/contract_integration_test.go`
- `internal/reedengine/naming_integration_test.go`
- `internal/reedengine/strandops_integration_test.go`
- `internal/reedengine/waitmark_integration_test.go`
- `internal/reedengine/watchdog_integration_test.go`

**Uses:**
- `internal/reedengine/state.go`
- `internal/reedengine/parse.go`

**Intent:**
D7, reed side: reed addresses its strand window by id instead of the session's current window, so a second window in the session (card 10's batten window) is never read as the strands' window.

`windowtarget.go` holds the one addressing seam, which replaces `exactSessionWindowTarget`:
it lists the session's own panes across all its windows (`list-panes -s -t =<session>` with `#{pane_id} #{window_id}`), and resolves, in order, the window of the recorded Selvage pane (`ReedState.SelvagePaneID`) when present in that listing, alive or a dead corpse; else the window of any present recorded strand pane; and only with none of those today's current-window target (`=<session>:`), which the seam keeps as its private fallback.
A recorded id counts as present only when it appears in that session-scoped listing, never through a server-wide pane target, since pane ids are server-wide and a stale id can name another session's pane; an absent id is treated as absent.
The resolved target is the window id (`@<n>`).
A listing failure is returned as an error, as a failed `list-panes` is today.

Every op that targets the session's current window today goes through the seam: pane enumeration, layout and split, and the window-size, resize-hook and attach-geometry reads.
`TmuxCmd.listPanes` takes the resolved target instead of the session name, and `reapSessionPanes`/`ReapSession`, which are engine-less, pass it in from their callers (the session-kill reap keeps the current-window fallback, since it ends the whole session).
The pane-generation read (`paneGenerationLocked`) stays session-scoped and outside the seam, targeting `=<session>` (`exactSessionTarget`), since it needs only session fields.
While the session has a present Selvage or strand pane, a batten window, current or not, is therefore never read as the strands' window and its pane is never reaped as untracked; with neither (a session's first `up`), the fallback applies and a Selvage pane created there defines the strand window from then on.
Update `doc.go`'s overlay and window-size sections for the strand window and the generation read's session target.

Tests:
- `windowtarget_test.go` (tier 1, `faketmux_test.go` extended to answer the session-wide listing apart from the per-window one): the Selvage pane's window wins; a dead Selvage corpse still resolves; with no Selvage, a present strand pane resolves; with none, the current-window fallback; a recorded id absent from the session's own listing (another session's pane) counts as absent;
- `contract_integration_test.go` (tmux tier): with a second window made current in the session, a reed op (layout, enumeration and the window-size read) leaves the strands untouched and the other window's pane unreaped; the pane-generation read targets the session;
- the existing tests that call `listPanes` or name the replaced target are updated to pass a target.

**ImpactSummary:** every reed op that enumerated or laid out the current window now targets the resolved strand window; behaviour is unchanged for a one-window session; the engine's exported surface is unchanged except the engine-less reap helpers' internal target.
