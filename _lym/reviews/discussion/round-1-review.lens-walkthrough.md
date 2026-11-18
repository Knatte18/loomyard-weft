```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision end to end, one concrete case each, and checked whether it meets its stated purpose.

**List**

- **D1**, a pawl run in a beam: `BeamUp(location)` runs inside the pawl run's Beam-Up row. `orchCLI.location` is the beam's worktree (`internal/orchcli/start.go:93`, `cli.go:209`), so `shedrun.ReadSeed(location, SelfRunID)` returns recipe `pawl`. The shared builder gives own-window + `KeepViewer` + `Title: pawl`. Purpose met.
- **D2 case 1**, fresh `lym beam start` after the pawl `lym loom start` put `loom-status` in the overview window. The operator is viewing that window and no other agent window is open.
  - The checklist reaches its own window and the sidecar keeps it on the next start. Both met.
  - The stated end state "viewer ends on the beam overview window" is **not met** (finding 1).
- **D2 case 2**, a reed restart leaves both strands dead. `runs` is not current, so it is replaced, and the dead `loom-status` is replaced with the beside spec. Purpose met.
- **D2 case 3**, `runs` live and current with `loom-status` live: early return, checklist untouched. Purpose met.
- **D2 failure: replace fails before removal** (refused in `validateNaming`, `validateDisplay` or `requireSessionLocked`, `internal/reedengine/replace.go:45-57`). A re-read finds `loom-status`, which the runs strand then replaces, as today. Met as stated.
- **D2 failure: replace fails after removal** (`addStrandLocked` fails, `replace.go:81-85`). A re-read finds no `loom-status`, so the runs strand is added. Met as stated.
- **D2 sidecar keep**: orchcli builds its identity with `statuspane.CurrentBuild` (`internal/orchcli/cli.go:241`), the same function loom uses, so `resolveStatusStrandAction` answers keep. Met.
- **D3**, own-window `loom-status` with `Title: pawl` placed apart. `createOwnWindowPaneLocked` hands `strandLabel(s.Name)` to `openOwnWindowLocked` on the ungrouped path and on the grouped placed-apart path (`internal/reedengine/spawn.go:167,186`). The break-out uses `strandLabel` (`ownwindow.go:405`). `validateDisplay` does not reject `Title` on an own-window anchor (`strand.go:70-76`). Swapping in `Title` on those paths names the window `pawl`. Purpose met. One gap in the rule (finding 2).
- **D4**, the beam's runs strand with pawl live. The frame reads the seed and polls status the same way `DobbyRunCells` already does (`internal/battencli/overview.go:260-300`), and the line names `pawl`, which matches the D3 window name. Purpose met.

**Findings**

1. **MEDIUM · design · `_lym/discussion/decision-record.md` D2 ("The viewer ends on the beam overview window, as after today's replace") and the criterion "After the move the viewer is on the beam overview window".**
   - **The step:** case 1 replaces an overview strand with an own-window spec, so `replacingWithOverview` is false. `killStrandPanes` is then called with `selectOnEnd=true` and moves a viewer on the ending window to `relevantWindow` (`internal/reedengine/replace.go:70-71`). That picks the newest live agent window, else the overview, else none (`internal/reedengine/ownwindow.go:456-470`). At that moment there is no overview window: the old one is ending and the runs strand is not added yet. So the viewer lands on an agent window such as orch's, or wherever tmux puts it.
   - **The new `loom-status` window** never takes the viewer, because of `KeepViewer` (`ownwindow.go:479`).
   - **The runs strand's `AddStrand`** never selects an overview window (`spawn.go:280-285`, and `doc.go` "An overview window is never selected here").
   - **Why "as after today's replace" does not carry over:** today's replace is overview-for-overview, and only that path selects the new window (`replace.go:97-98`).
   - **Result:** D2 names an end state that no step of its mechanism produces. Plan would have to discover that it needs a reed selection step: a new op, a `FollowRun`-like call, or an overview-select on add when the viewer was moved. The decision should state that mechanism, or drop the claim and its criterion.
2. **LOW · consistency · `_lym/discussion/decision-record.md` D3 ("A strand in a shared window group ignores `Title`").**
   - A strand that declares a window group but is placed apart (group full or over `window_share_max`) opens its own window through the same `strandLabel` path as an ungrouped strand (`internal/reedengine/spawn.go:185-186`).
   - D3 calls it both "placed apart" (so `Title` names its window) and "in a shared window group" (so `Title` is ignored). The criterion covers only "a strand of a shared group in the group's window".
   - D3 should state which rule applies to a grouped strand placed apart.

**Tool calls**

- `spawn.go` `openOwnWindowLocked` callers, select sites and `orchCLI` fields: the manifest lacked which name each own-window path hands to `openOwnWindowLocked`, and the orchCLI wiring for `location`.
- `spawn.go:160-200` and the `location` wiring in orchcli `start.go`/`cli.go`: to confirm the grouped placed-apart path and that `BeamUp` runs against the beam's worktree location.
- Render and `validateDisplay` reads of `Title`, plus orchcli's `currentBuild` source: to check whether reed refuses `Title` on an own-window display and whether both callers write the same build identity.
