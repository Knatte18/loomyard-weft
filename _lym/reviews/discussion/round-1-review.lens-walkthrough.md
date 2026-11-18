```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**
- **D1**: A pawl run in beam B, state `running`, reaches its Beam-Up row. `orchcli.BeamUp(location)` calls `ensureRunsStrand`, which finds a live `loom-status` overview strand and no runs strand. `ReplaceStrand` then gives the overview window to the runs strand, the runs sidecar is written, and the builder re-adds the checklist beside the runs strand with group `pawl`. Placement is `placeNewShared` at size 1 with share max 2, so the window is named `pawl` and does not take the viewer.
  - Happy path: purpose met.
  - Failure path: purpose not met (finding 1).
- **D2**:
  - Bootstrap case: loom start with a runs strand present. The seed is written by `seedAndCommitBootstrap` before `ensureStatusStrand` runs (`internal/loomcli/start.go:524-553`), so the recipe is readable when the builder runs. Purpose met.
  - D1 case: the builder is asked for beside-runs after the replace. Purpose met.
  - The builder's input set is "placement" only, but it also needs the location whose seed it reads. That location is unstated, but the step inputs are available on both paths, so this is not filed.
- **D3**: The beam-B overview pane resolves `acting` to B and reads B's `SelfRunID` seed (recipe `pawl`) and status (state running). `overviewStage("pawl", row)` goes through `loomrecipe.Rows`, which serves a loom recipe. The window-name step then reads reed state.
  - Purpose not met for the window name (finding 2).
  - Conflicts with a Constraint (finding 3).

**Findings**

1. **MEDIUM · design** · `_lym/discussion/decision-record.md`, D1
   - **Summary**: An early return between the replace and the re-add loses the checklist, and the way forward that warning names (`lym beam start`) never brings it back.
   - **Walk**: `ReplaceStrand` removes and kills the old strand before it adds (`internal/reedengine/replace.go:65-85`). A failed add therefore leaves neither strand. A failed runs-sidecar write also returns before D1's re-add (`internal/orchcli/runs.go:98-100`). Both warnings tell the operator to run `lym beam start` (`runs.go:107`). On that rerun `ensureRunsStrand` takes the `default` (AddStrand) or `runs != nil` (replace) branch, and D1 re-adds only in the `loomStatus != nil` branch (`runs.go:84-95`). So the pawl checklist stays gone while the run lives, which contradicts the Goal.
   - **Fix**: State whether the re-add also runs after a failed runs-sidecar write. Have these warnings name `lym loom start`, or re-add whenever the runs strand ends up present with no status strand and a self run not done.

2. **MEDIUM · design** · `_lym/discussion/decision-record.md`, D3
   - **Summary**: The window name D3 reads "from the session's reed state" is not in reed state, so the step's input differs from what reed hands it. The rationale's `window_share_max` 0 case breaks exactly where it claims to hold.
   - **Walk**: `Strand` persists `Display.Window`, the group, but no window name (`internal/reedengine/state.go:21-53`). The name is set on the tmux window at launch:
     - by the group when it is shared (`internal/reedengine/spawn.go:197`);
     - by the role via `strandLabel` when placed apart or ungrouped (`spawn.go:167`, `spawn.go:186`).
     - With share max 0, reed.json still holds `Display.Window: "pawl"` while the window reads `loom-status`. A reader taking the group from reed.json names a window that does not exist.
     - Getting the right name means recomputing reed's rule from `WindowGroups[group].Shared`, or asking tmux. That is the recompute the rationale says D3 avoids.
   - **Fix**: Name the reader and the source: tmux's `window_name` for the strand's pane, or `Display.Window` plus `WindowGroups[...].Shared` else the role. Correct the rationale accordingly.

3. **LOW · consistency** · `_lym/discussion/decision-record.md`, Constraints ("The overview reads files only, takes no lock")
   - **Summary**: Read through reed's exported reader, D3's reed-state step takes a lock.
   - **Walk**: `reedengine.LoadState` reads through `state.ReadJSON`, which acquires a shared read lock on `reed.json.lock` (`internal/reedengine/state.go:150-156`, `internal/state/state.go:96-102`).
   - **Fix**: Name an unlocked read for D3, or reword the Constraint.

**Tool calls**
1. `internal/battencli/overview.go:60-135`: to check that `overviewStage` accepts a pawl/loom recipe; the manifest lacked its body.
2. Grep of `internal/loomcli/start.go` for seed writes: to order the seed write against `ensureStatusStrand`; the manifest lacked `start.go`'s verb body.
3. `internal/loomcli/start.go:500-560`: to confirm the seed is written before the status strand is added on the D2 bootstrap path.
