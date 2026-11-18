```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism step by step on one concrete case and checked the end state against the decision's stated purpose.

**Decisions walked**
- **D1**, purpose not met as written. Case: the beam session holds a live `loom-status` strand in the overview window and no runs strand, and `lym beam start` runs.
  1. `ReplaceStrand(loomStatus.GUID, runsSpec)` kills the status pane, so the overview window now holds `runs`.
  2. D1 then calls D2's builder for the status spec, but the only strand list it holds is the one it read before the replace. That list has no runs strand in it. `statusStrandAddSpec` today picks its anchor from that list, so the builder returns the overview anchor.
  3. `AddStrand` then fails, because reed refuses a second overview strand. The result is a warning and no checklist.
  - The record only says "the beside-runs spec of D2". It never says the builder takes an explicit beside-runs input, or that D1 must read the strands again after the replace. That gap is finding 1.
- **D2**, purpose met only under a config condition. Case: the `pawl` recipe, `Display{AnchorOwnWindow, KeepViewer, Window:"pawl"}`.
  - `planOwnWindowPlacement` sets size = 1 and `Shared = size &lt;= window_share_max`. The default `window_share_max` is 2, so the strand gets a new shared window named `pawl`.
  - With `window_share_max: 0`, placement is `placeApart`, and `openOwnWindowLocked` names that window `strandLabel(s.Name)`, which is `loom-status`.
  - So the window name depends on a reed config key the record never mentions. That is finding 2.
- **D3**, purpose met except in that same config case. Case: the beam's self run is seeded with `pawl`, status `failed`, at row X.
  - `ReadSeed(location, SelfRunID)` returns `pawl` and the status read returns `failed`. The line shows `pawl`, the stage and the window name `pawl`.
  - With `window_share_max: 0`, the line names a window that does not exist (finding 2).
  - The record calls the run "halted" and "finished", but shed has no such states. They need mapping to real ones, which is finding 3.

**Findings**
1. MEDIUM, D1/D2: D1's second step gets the wrong input.
   - Built from the strand list `ensureRunsStrand` read before the replace, the builder gets a list with no runs strand, returns the overview anchor, and reed refuses the add.
   - Fix: D2's builder takes an explicit beside-runs input, or D1 reads the strands again after the replace. An acceptance criterion should pin that the re-added spec is own-window.
   - Evidence: `statusStrandAddSpec` decides by looking for `orchcli.RunsStrandName` in the list passed to it, and `ensureRunsStrand` reads `strands` once, before `ReplaceStrand`.
2. MEDIUM, D2/D3/Constraints: "reed names the window for the recipe" holds only when `window_share_max` ≥ 1.
   - With 0, reed takes `placeApart` and names the window `loom-status`, so D3's line points at a window that does not exist.
   - Fix: either state the dependency and its pin (`planOwnWindowPlacement` in `internal/reedengine/ownwindow.go`, the size-1 group shared when 1 ≤ `window_share_max`, default 2 in reed's templates), or have D3 print the window name reed really used, or the role, rather than the group name.
   - Also unverified: does a `KeepViewer` strand that opens a new shared window still not take the viewer? Today `KeepViewer` is only known to work on the `placeApart` path, through `newAgentWindowSelected`.
3. LOW, D3/Auto-mode assumptions: "halted" and "finished" are not shed states.
   - `shedengine` has `running`, `paused`, `done`, `blocked`, `failed` and `awaiting`.
   - Fix: name `StateDone` as the one finished state, and show every other state, including `blocked` and `failed`, so the case the operator advances by hand is covered.
4. LOW, D1: on a re-add the record still holds.
   - `resolveStatusStrandAction` keeps a live strand whose sidecar GUID and build match. D1 writes the new GUID with `c.currentBuild()`, and the bootstrap compares against `statuspane.CurrentBuild()`.
   - Both have to produce the same `BuildIdentity`. That is a plan-time check, not a design flaw.

**Tool calls made**
- Read `validateDisplay` and `killStrandPanes` in reed: I needed the replace and kill behaviour and the rule on window groups.
- Read group-window naming in `spawn.go`, `render.ValidateDisplay` and the `orchCLI` fields: I needed how a group names its window, and whether orchcli holds a location from which to read the seed.
- Grepped `SelfRunID` in loomcli, `planOwnWindowPlacement` and the shed `State` constants: I needed to confirm the pawl seed's run id, the share rule, and which run states exist.
- Read the `placeApart` naming in `spawn.go` and the default `window_share_max`: I needed what decides whether the window gets the group's name.
