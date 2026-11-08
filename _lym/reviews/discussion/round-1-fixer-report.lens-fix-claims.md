```yaml
lens: fix-claims
outcome: reported
```

**Fix-claims lens:** I checked every claim about the code that the fix added or rewrote against HEAD.

**Finding: MEDIUM**
- **Changed text:** the D4 Bound in `_lym/discussion/decision-record.md`: "That route is bounded by the key's guard category, which the local config layer refuses, so a run's own worktree cannot turn it on".
- **Summary:** The guard category keeps the key out of the local layer only. The worktree's own tracked config layer is a separate layer, and the guard category does not keep the key out of it. So "a run's own worktree cannot turn it on" is a bound the code does not hold.
- **Evidence:**
  - `internal/configengine/layers.go:40-46` defines `LayerWorktree` ("the worktree's own tracked file under its anchor") apart from `LayerLocal` ("the untracked local override file under the anchor's .lym").
  - `internal/configengine/doc.go:96-102`, "The local rule": "The local layer holds only keys classified [CategoryNone]". That rule names the local layer alone; nothing in the code I read refuses a guard key in the worktree layer.
  - `internal/boardcli/runsettings.go:24-26`: the board entry's `run_settings` check "guards a mistaken write, not a determined one". That is a second route for setting a loom key on the run that the guard category does not close.
  - A defensible restatement would say only what holds: the key is guard-category and defaults to false, and Raddle-File's checks and the draft status still apply.

**Claims that hold:**
- D6: the cleared field list matches `sketchOf` (`internal/boardengine/sketch.go:58-65`).
- D6: `endsAsSketch`'s equality is as described.
- D6: the filing mark under `shedrun.RunDir` sits beside status.json and child-recipe, and `removeRunRecords` removes `RunDir` (`cleanup.go:391`). The raddle store sits beside board.json, and nothing in cleanup removes it.
- D8: batten cancel's claim clear calls `hubBoard.SetStatus` (`internal/battencli/cancelcmd.go:319`).
- D10: `NewDependencies` counts every task whose `project` names the project as a member, whatever its status, so draft members are landing members (`dependency.go:23`).
- D2: the auto-accept key name `<perch>_auto_accept` and its guard category match `internal/reviewbudget/reviewbudget.go:28,42`.

**Tool calls:**
1. `internal/configengine/doc.go` local rule and `cancelcmd.go:305-325`: I needed the guard category's actual scope and cancel's status write, which my context did not hold.
2. grep in `layers.go` for local layer and `RunSettings` users: I needed to know whether layers other than the local one exist.
3. `layers.go:28-48` and grep in `runsettings.go`: I needed the definitions of the worktree and local layers.
4. `internal/boardcli/runsettings.go:1-40`: I needed to know whether `run_settings` is a guarded route.
