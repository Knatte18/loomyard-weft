# Round 5 fixer report

All eight findings validated against the code and fixed.
Checked: `paneIDsByTop` (`internal/reedengine/apply.go`) sorts by `pane_top`; `AttachArgv` (`attach.go`) lists panes and plans before the attach; `resizePinHookArgvs` (`windowsize.go`) emits one standalone `resize-pane -y` per hook entry.

## Fixed

- F1 (MEDIUM): D1 states every pane list a layout or pin is planned from is read on the unzoomed window, and the attach pre-flight unzooms, lists and re-zooms under its op lock; the zoom acceptance criterion now zooms a strand that is not the top pane.
- F2 (LOW): D1's trigger is every reed step that changes the strand window's panes, layout or active pane, read at step start and re-zoomed also when a guard ends the step early; the Rationale lists the unzooming commands.
- F3 (LOW): D1 states that the `window-resized` hook and the attach chain carry the bracket as run-time tmux commands (a first entry records the zoom in a window user option, an entry after the pins re-zooms the active pane); D7 appends the pins to the attach chain after its `select-layout`.
- F4 (LOW): the `PATTERN-told-geometry` Constraints bullet names the sorted list of names and revive functions, without start times.
- F5 (LOW): D12's booter re-run carries the same skip as a revive function's up, so the trigger is evaluated once.
- F6 (LOW): D15 creates the temporary file exclusively, a collision is a skip, cleanup removes only a file this run created, and the Bound no longer claims a killed run leaves nothing.
- F7 (LOW): Scope reads "when a worktree's recorded session is revived".
- F8 (NIT): D12's Bound says "for which the revival predicate holds".

The support log gained a Round 5 entry.

## Disputed

None.

## Deferred

None.
