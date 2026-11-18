```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code that already writes, reads or replays the state each decision changes.

**Paths traced**

- **D1 (shared spec)**
  - `ensureStatusStrand`'s keep, replace and add flow in loom start: survives, since the placement rule is unchanged and the beside case is built from the shared spec.
  - The llm-arm status-strand removal (`0cca91e0`): unaffected.
  - The `resolveRecipe` source of `c.recipe`: breaks in one case. It defaults to tapestry when the run has no seed yet (finding 2).
- **D2 (move, then add runs)**
  - The Beam-Up row re-running on resume: survives. The runs strand is present and current, so the call does nothing.
  - Beam-Await restarting the dobby through `orchcli.BeamUpWorktree`: survives, through the same no-op.
  - `lym beam start` after a binary update: survives. The runs strand is replaced and `loom-status` is untouched.
  - The next `lym loom start`, which reads the sidecar: survives if orch's `currentBuild` matches `statuspane.CurrentBuild`.
  - `ReplaceStrand` replacing the overview strand with an own-window spec: survives. It ends the overview window, keeps the slot, then the add of the runs strand finds the overview free.
  - A reed restart where both strands are left dead: breaks (finding 1).
- **D3 (titled own window)**
  - New-window placement in `spawn.go` (`createOwnWindowPaneLocked`'s ungrouped and placed-apart branches): covered by the record.
  - The break-pane out of the shell window in `ownwindow.go`: covered by the record.
  - Grouped windows are named by their group: the record matches.
  - The watchdog name repair in `namerepair.go` touches pane titles only: survives.
  - Window order and the shell window's detection by the name `shell` (`windoworder.go`): survives for today's recipe names (finding 4).
  - The window-bar buttons show `#{window_name}`: they read the new name correctly.
- **D4 (own-run line)**
  - `DobbyRunCells.poll` reads the beam's child as `SelfRunID` in the pair worktree, so the pawl run is the beam worktree's own `self` run. Reading that run is consistent.
  - The prime's dobby runs are keyed by slug, not `self`, so they don't trigger the line. Whether the prime has a `self` seed at all, I did not check.

**Findings**

1. MEDIUM, D2 and Goal: after a reed server restart, nothing restores the checklist.
   - Evidence: `resolveStatusStrandAction`'s doc says a reboot, crash, kill-server or zombie-boot reap leaves dead entries in the table. Both `runs` and `loom-status` stay tracked but dead.
   - `ensureRunsStrand` then takes the `runs != nil` branch and replaces only the runs strand. D2's move runs only when no runs strand exists, and `ensureStatusStrand` runs only from `lym loom start`.
   - So `lym beam start` brings the overview back without the checklist, while the Goal promises the checklist "while a beam's pawl run lives".
   - The record is silent on this. The open risk names only older-build sessions.
   - Fix: also re-add or replace a dead `loom-status` strand in D2 while the worktree's run is live, or narrow the Goal and record the gap as a risk.
2. MEDIUM, D1 and D3: loom start's title source can give the wrong name, and the keep path never corrects it.
   - Evidence: `resolveRecipe` in `internal/loomcli/arm.go` sets `c.recipe` to tapestry when the run has no seed yet. D1 says the beside spec's `Title` is "the run's recipe name" but does not say loomcli reads it from the seed, as D2 does for orchcli.
   - If the strand is added before the seed is readable, or with the default, the window is named `tapestry`.
   - Once the sidecar matches, the keep path never re-specs the strand, so the wrong name sticks.
   - Fix: D1 should state that both paths take the title from `shedrun.ReadSeed`, with no tapestry default, and leave it empty when no seed is found.
3. LOW, D2: where the operator's view lands after the move is unstated.
   - Evidence: `ReplaceStrand` ends the overview window while the viewer may be on it. The new own window has `KeepViewer` and the replacement is not an overview strand, so nothing selects a window.
   - The following add of the runs strand, an overview strand, may not take the viewer either.
   - The record does not say where the viewer is left. This only affects what is shown.
4. LOW, D3: a title can clash with a window name reed keys on.
   - Evidence: `windoworder.go` names the shell window `shell` and judges a repair by `shell.Name != shellWindowName`. `overviewWindowName` is `overview`.
   - A seed recipe named `shell` or `overview` would produce a confusing window. No current recipe has such a name, and the record states no bound on titles.

**Tool calls**

1. `ReplaceStrand` in `replace.go` and the window-naming call sites (`strandLabel`, `overviewWindowTitle`): my context did not show reed's replace semantics or every site that names a window.
2. `replace.go` and `spawn.go` around own-window creation: needed the slot keeping, the ended-window handling and the placement branches.
3. `windoworder.go` and window-name lookups (bar formats, `bindings.go`): to find code that identifies windows by name.
4. Name-based checks and other `loom-status` readers outside the cited files: to find other readers of the role or window names.
5. `namerepair.go` and `rename-window` uses: to check whether a background repair renames windows and so undoes D3.
6. `resolveRecipe` and `armAt` in `loomcli`: to check what `c.recipe` holds when the status strand is added.
