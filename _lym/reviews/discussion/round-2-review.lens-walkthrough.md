```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision end to end on one concrete case, passing each step the data the step before it produces.

- **D1:** held. Case: a beam session where the pawl run is running at its "bring up the beam's dobby and orch" row, the loom status strand sits in the overview window and there is no runs strand. Purpose met.
  - The pawl step's `BeamUp(location)` gives `wireCLI` the beam location, then `ensureRunsStrand` runs: it finds `loomStatus` and no `runs`, `ReplaceStrand` gives the runs strand the overview slot, and the runs sidecar is written.
  - The self status reads `running` (not `StateDone`). The D2 builder is asked for beside-runs and the seed's recipe is `pawl`.
  - `AddStrandUnless(spec, "loom-status")` finds no holder, since the replace just removed it, so it adds the strand.
  - `createOwnWindowPaneLocked` records `pawl` as size 1, and `1 <= window_share_max` (default 2), so the group is shared. That gives `placeNewShared` and `openOwnWindowLocked(st, s, "pawl", "pawl")`, which opens a window named `pawl`.
  - `KeepViewer` means `newAgentWindowSelected` returns false, so the viewer stays where it is.
  - The sidecar goes to `statusSidecarPath` under `loomengine.LoomScratchDir`, which orchcli already imports. The build comes from `c.currentBuild`, which is `statuspane.CurrentBuild` (`internal/orchcli/cli.go:241`), the same identity `resolveStatusStrandAction` compares against. So the next `lym loom start` keeps the strand.
- **D2:** held. Case: `lym loom start` resumes in a beam whose runs strand is present and that has no status strand. Purpose met.
  - `seedAndCommitBootstrap` writes the seed before the strand work, so the seed holds `pawl` when `ensureStatusStrand` reads it.
  - The strands hold `runs`, so the builder gives beside-runs: own-window, `KeepViewer`, group `pawl`.
  - `AddStrandUnless` adds the strand, or skips it if the race was lost.
  - The replace path (a dead own-window status strand with group `pawl`) re-records the same size-1 group, and `planOwnWindowPlacement` raises no size conflict.
- **D3:** held. Case: the runs strand's pane runs `lym batten overview` while pawl is running. Purpose met, apart from the window-name gap in the finding below.
  - The pane cwd is the reed geometry's `PaneCwd`, which equals `AnchorPath` in hub mode, so `acting` is the beam.
  - `ReadSeed(acting, SelfRunID)` gives `pawl`, the status gives the state and `CurrentProducer`, and `overviewStage` labels the row through `loomrecipe.Rows("pawl")`. When the row has no number it falls back to the bare row.
  - `LoadState(<anchor>/.lym)` reads the same file reed's `stateDir` writes (`internal/reedengine/lifecycle.go:36`). The strand matched by role `loom-status` has `Display.Window` `pawl`, and `WindowGroups["pawl"].Shared` is true, so the line names `pawl`. That equals the window name `openOwnWindowLocked` was given.

Findings:

1. **LOW, consistency** — `_lym/discussion/decision-record.md#D2` (and the Goal line). At `window_share_max` 0 the window D1 and D2 produce is named `loom-status`, not the recipe.
   - Evidence: `planOwnWindowPlacement` records `Shared: size <= shareMax`, which is false at 0, and returns `placeApart` (`internal/reedengine/ownwindow.go:215-218`). `createOwnWindowPaneLocked` then names the window `strandLabel(s.Name)` (`internal/reedengine/spawn.go:185-186`).
   - D3's Rationale admits this case, but D2 claims the group names the window, and the Goal promises a window "named for the run's recipe" without that exception.
   - Fix: D2 states that the recipe names the window only while `window_share_max` is at least 1, and that at 0 reed names it `loom-status`.
   - This is not BLOCKING, so under this round's incremental rule the handler may reject it as out of bounds.

Tool calls:
1. Grepped `internal/orchcli` for `currentBuild` and `executable`, `internal/battencli/overview.go` for `overviewEntryNumber`, `internal/reedengine/spawn.go` for the `openOwnWindowLocked` signature, and `internal/loomengine` for `LoomScratchDir`. My context lacked orch's build source, which decides whether the sidecar D1 writes passes loom's keep check. It also lacked how pawl's row is numbered, which window name the shared path opens, and where the status sidecar lives.
