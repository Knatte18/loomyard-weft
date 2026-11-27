```yaml
lens: fix-proposal
outcome: reported
```

Six of the reviewer's proposals the fixer carried out hold up when challenged; three leave defects:

1. **MEDIUM** — F17, card 12's bound: "it writes only the told hub's places file, … the hub's log files, and the files beside the per-user script lock".
   The verb writes one more thing: card 8's `run` step 3 writes every script "to a temp file through `lymtemp.CreateTemp`", and D3 says "the script file is a temp file under `lymtemp`".
   So the bound states a limit the code will not hold. It needs to name the temp script file under `lymtemp`'s directory.

2. **LOW** — F10, card 13: "`TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing` goes into the existing `tier1` file `internal/reedcli/placewindow_integration_test.go`".
   That file's header comment reads "placewindow_integration_test.go runs placeTerminalWindow against a real hub and a directory outside any worktree". Once it also holds a `handTerminalWindow` test, the comment is stale, and card 13 names no change to it.

3. **LOW** — F7, card 12: the `RefusedTaskMode` row writes `window_placement: sideways` into the task worktree's beam config file. It does not get a hub of its own, but `RefusedMachineMode` does.
   If `TaskWindow` runs after it on the shared hub, that task's mode resolution is refused and the verb takes the resolution-failure path. `TaskWindow`'s assertions would still pass: the marker appears, the record is registered by `Spawn`, and the entry is settled. The row would then no longer exercise the flow path it claims to test.
   `RefusedTaskMode` needs its own hub or task, or must restore the file.

I also tried to break F12, which replaces two member targets with `internal/reedcli/window.go#` and drops the re-sign arrow. The only caller of `openTerminalWindow` is in `window.go` itself (`window.go:123`), so the file target still covers it and nothing loses the caller check.

Tool calls:
- One grep for `openTerminalWindow` callers and `configengine.validateOverride`: my context did not show whether dropping F12's re-sign arrow leaves a caller uncovered, or whether a machine-layer `Set` validates values (it only showed the machine rule check, which I could not settle without spending more calls).
