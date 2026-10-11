# Card 3 — reed name repair behind the activity check and the paste-submit

**Edit:**
- `internal/reedengine#Engine.repairSessionName`
- `internal/reedengine/namerepair_test.go`
- `internal/reedengine/doc.go`

**Uses:**
- `plan:internal/reedengine#Engine.operatorActiveLocked`
- `plan:internal/reedengine#Engine.pasteSubmitLocked`
- `internal/reedengine#SessionNamer.SessionIdle`
- `internal/reedengine#SessionNamer.RenameText`

**Intent:**
Make the watchdog's session-name repair, the one pane writer that cannot call shuttle, use the same two reed ops as shuttle (decisions D1 and D6).

`repairSessionName` keeps its drift check and its capture plus `SessionNamer.SessionIdle` reading.
After an idle reading it asks `operatorActiveLocked(s.PaneID)`: an active operator logs at Debug ("operator active, deferring session name repair", with the quiet window) and returns false, so the next watchdog tick retries; a query error logs at Warn and does not hold the rename (A4).
It then types the rename with one `pasteSubmitLocked(s.PaneID, namer.RenameText(s.Name), true)` in place of today's `send-keys -l` plus a separate `send-keys Enter`; a failed paste logs at Warn and returns false as today.

Bound: on a query error the rename is still held by the namer's idle reading, which requires an empty input box, so the fail-open can type only past an operator who is active in the pane but has not typed into the box yet (R2's window).
The repair uses the Enter variant of the paste-submit, so it relies on Claude Code submitting a slash command on an Enter right after a bracketed paste end, which card 10's live check pins with `/color`.
If that check ever fails, the leftover `/rename` sits in a non-empty box, the namer's idle reading then reads not idle, and the repair never types into it again; the text stays visible to the operator rather than merging with a draft.

Tests in `namerepair_test.go`, as rows of `TestRepairNames_SessionName` over its scripted tmux: an attached client with fresh activity on the pane makes the pass type nothing and report no repair; an idle pane with no active client makes exactly one tmux invocation carrying `set-buffer`, `paste-buffer -p -d` and `send-keys … Enter` with the rename text, and no `send-keys -l`; a failing `list-clients` still types the rename.
Update the name-repair paragraph of `internal/reedengine/doc.go`.

**ImpactSummary:** One function in reed's watchdog changes how it types; reedcli's wiring of the namer is untouched.
