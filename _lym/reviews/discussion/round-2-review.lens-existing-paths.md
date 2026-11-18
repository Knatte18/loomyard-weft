```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each existing path that writes, resets or reads the state D1–D3 introduce or change.

**D1: re-add after the runs strand takes the overview window**
- Later `lym loom start` or resume, `ensureStatusStrand` → `resolveStatusStrandAction` (`internal/loomcli/bootstrap.go:226`): **survives.** D1 writes the status sidecar with the new guid and the current build, so `StrandCurrent` keeps the strand. A build change replaces it, and D2's beside-runs pick puts it back in its own window.
- Later `lym beam start` → `ensureRunsStrand` (`internal/orchcli/runs.go:81-96`): **survives.** A current runs strand returns early. A stale one is replaced alone. The loom status branch is not reached while a runs strand exists.
- `ReplaceStrand` overview selection (`internal/reedengine/replace.go`, `endedCurrent && replacingWithOverview`): **survives.** It selects the runs window, and D1's `KeepViewer` add does not take the viewer (`ownwindow.go:475-483`).
- Reed revival or resume relaunching persisted strands → `createOwnWindowPaneLocked` (`spawn.go:164-206`): **survives.** The group record is kept while the strand names it, so the window is rebuilt under the same name.
- Bootstrap replace of a stale status strand while another strand changes (`sharedbootstrap.go:302-310`): **survives.** A replace that fails because the guid is gone logs a warning and returns nil. D1's own re-add restores the checklist.
- `BeamUp` callers: beamcli, the pawl step in `loomcli/pawlwiring.go:107`, adoptcli. All pass the beam location to `wireCLI`, so the seed, status and sidecar are the beam's: **survives.**
- Prime's `ensureRunsStrand`: the prime holds no loom status strand and no self run, so nothing is added: **survives.**

**D2: shared builder and window group named for the recipe**
- Seat tables set the only other `Display.Window` groups (`internal/seatengine/prompt.go:183,189-197`). They are named by role prefix and round, never by recipe, so a `pawl` group collides with nothing: **survives.**
- Window renaming: reed renames only the shell window (`internal/reedengine/windoworder.go:204-206`). A group-named window keeps its name through apply and reorder: **survives.**
- Group-record pruning and a later `window_share_max` edit: the record keeps its first strand's choice until the group empties, and a replace re-decides it. D3 reads that same record: **survives.**
- Import graph: none of reedengine, render, shedrun, shedengine or selfexec imports statuspane, and shedrun imports only lymcwd and lymdirs. A statuspane builder adds no cycle: **survives.**

**D3: overview line for the worktree's own run**
- `overviewCmd` resolves `acting` from the runs pane's cwd, which is reed's `PaneCwd`, the anchor in hub mode. The reed state it reads is the same `AnchorPath/.lym/reed.json` that `stateDir` writes (`lifecycle.go:36-38`): **survives.**
- `reed.json` writes go through `state.WriteJSON` under the lock that `LoadState`'s `ReadJSON` takes shared, so the overview never sees a torn file: **survives.**
- Self run directory: in a beam, `SelfRunID` resolves to the beam slug's directory, distinct from the batten runs keyed by task slug. In the prime there is no self seed, so no line: **survives.**
- `StateAwaiting` (`shedengine/status.go:23`) is not done, so it shows, which matches "every state but done": **survives.**
- Only `ensureStatusStrand` reads the status sidecar, so D1's write is read correctly: **survives.**

**Findings:** none. Every traced path keeps or correctly reads each decision's result. This round is incremental, so a new finding needs BLOCKING severity on a changed hunk, and no path breaks one.

**Tool calls**
1. Searched for users of `LoomStatusStrandName`, `statusStrandDisplayName` and `statusSidecarPath`, and for window-group setters: to find other writers and readers of the loom status strand and its sidecar, which the manifest did not list.
2. Searched reed for `rename-window` and automatic-rename, and for `Window` group setters in shuttle, render, seat and loom engines: to check whether any path renames a group-named window or claims a recipe-named group.
3. Read `tableWindow` in `internal/seatengine/prompt.go`: to see how the one other group setter names its groups.
