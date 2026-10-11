# Card 2 — reed operator-activity query and operator_quiet_s

**Create:**
- `internal/reedengine#Engine.OperatorActive`
- `internal/reedengine#Engine.operatorActiveLocked`
- `internal/reedengine#activeClientOnPane`
- `internal/reedengine#operatorQuiet`
- `internal/reedengine#defaultOperatorQuietS`
- `internal/reedengine/activity_integration_test.go`

**Edit:**
- `internal/reedengine#Config`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine/io_test.go`
- `internal/reedengine/config_test.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#Engine.resolvePaneInThisSessionLocked`
- `internal/reedengine#TmuxCmd.output`
- `internal/reedengine#exactSessionTarget`

**Intent:**
Add reed's operator-activity query (decisions D1 and D3) and its `reed.yaml` key.

`Config` gains `OperatorQuietS int` under the yaml key `operator_quiet_s`.
Both `template_posix.yaml` and `template_windows.yaml` gain `operator_quiet_s: 5` with a comment: seconds since an attached client's last activity on a strand's pane within which lyx types nothing into that pane; an already-materialized `reed.yaml` lacking the key takes the default until `lyx config reconcile` writes it.
`operatorQuiet(cfg)` returns `OperatorQuietS` seconds, floored to `defaultOperatorQuietS` for a zero or negative value, so a hand-built or older Config still gets 5 s.

`Engine.OperatorActive(guid)` takes the op lock, requires the session, resolves the pane like every transport op, and calls `operatorActiveLocked`.
`operatorActiveLocked(paneID)` runs `list-clients -t =<session> -F '#{pane_id} #{client_activity}'` through `e.tmux.output` and answers `activeClientOnPane(out, paneID, time.Now(), operatorQuiet(e.cfg))` and the quiet window, so a caller can name the window in its hold reason.
`activeClientOnPane` is pure: it reads one client per line, `#{pane_id}` being that client's active pane in its current window (assumption A6), and reports whether any line names `paneID` with a `client_activity` epoch second later than `now` minus `quiet`; a malformed line is skipped.
No attached client, or clients on other panes only, answer not active.
A failed `list-clients` is returned as the error; the callers (card 3, card 5) log it at Warn and do not hold the write (A4).
`operatorActiveLocked` exists so the name repair, which holds the op lock, can ask without re-taking it.

Tests:
- `io_test.go`: a table over `activeClientOnPane` with a fixed `now`: a client on the pane with activity 1 s ago is active, one 6 s ago is not, one on another pane is not, an empty list is not, a malformed line is skipped.
- `config_test.go`: `TestLoadConfig_TemplateDefaultsResolve` asserts `OperatorQuietS` is 5, and a row asserts `operatorQuiet` floors 0 and -1 to 5 s.
- `activity_integration_test.go` (`//go:build tmux && linux`, reusing `startInPTY` from `attachgeometry_integration_test.go`, which shares the constraint): a real client attached in a pty to the session's strand pane, after a keystroke written to the pty, reads active for that pane and not active for a second pane of another window; this pins A6.

Raise `internal/reedengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level tests, and document the query and the key in `internal/reedengine/doc.go`.

**ImpactSummary:** One Config field and one template key in reed; the new query has no caller until cards 3 and 5.
