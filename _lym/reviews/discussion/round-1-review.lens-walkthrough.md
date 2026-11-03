```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

- D1, met. Case: a beam is created. The pawl run's first `lym loom start` adds the status strand as an overview with key 1, in a session that holds only the shell window, so it opens before the lowest-index window, at index 0. Then `lym beam start` adds the runs strand with key 0. No overview has a key at or below 0, so the runs window opens before the status window. End order: runs, status, agents, shell.
  - Relaunch, keys 0 and 1: `launchStrandLocked` keeps the old window listed until `killReplacedOwnWindowPaneLocked` runs after the create (`internal/reedengine/spawn.go:90-97`). The new runs window lands right after the old runs window and before the status window, the old one is killed, and the order holds.
  - Relaunch, tied keys A and B at 0: relaunching A places it after B, which matches "ties sit in the order they last opened".
  - Data for the key: `openOwnWindowLocked` holds both `st` and `windowByPane` (`spawn.go:261-283`), so each listed window's key can be read from reed state.
- D2, met. Case: a beam session with runs at index 0, status at index 1, the orch agent, and the shell. `relevantWindow` gets windows that carry `Index` (`windoworder.go:34-42`). `followTargetWindow`'s listing carries `#{window_index}` (`follow.go:55`). Both inputs hold what is needed to pick the lowest-index overview. The orch agent window is live, so it still wins, and with no live agent the fallback lands on runs, the beam overview.
- D3, met. Case: a pawl run started by a batten pass. The pass seeds the child run with pawl before it spawns `loom start --no-attach` in the beam's anchor (`internal/battencli/wire.go:524`, `wire.go:1286`). `resolveRecipe` therefore reads pawl (`internal/loomcli/arm.go:127-135`), and the window is titled `pawl` with key 1. Task-session case: no seed, so `resolveRecipe` gives tapestry and `seedAndCommitBootstrap` writes tapestry through the same `resolveSeedRecipe` (`internal/loomcli/sharedbootstrap.go:105`, `206-211`). The title is `tapestry`, and the status window is the only overview.
- D4, met. Case: the status strand holds an overview window at index 0 with key 1, and `lym beam start` runs. With the loom-status branch dropped from `ensureRunsStrand`, there is no runs strand, so `AddStrand` adds one with key 0 and no cap refuses it. It opens before the status window, and overviews are never selected on open (`spawn.go:280-286`). The status strand is untouched. A stale runs strand goes through `ReplaceStrand` (`replace.go:64-99`), which reopens it with key 0, still first.
- D5, met. Two cases:
  - Status strand in an own window with `KeepViewer`, and the new lym installed. `StrandCurrent` is false because the sidecar build differs (`internal/statuspane/build.go:80-82`), so `resolveStatusStrandAction` returns replace. `ReplaceStrand` removes the old strand, adds an overview with key 1 placed after the runs strand (whose stored key reads as 0), and selects the new window if the viewer was on the old one (`replace.go:70-99`).
  - Status strand gone, because the runs strand replaced it. `findStatusStrand` misses, so the strand is added.
  - Who runs that start: the batten pass's loom spawn does cover pawl (`wire.go:524`, `1286`), and a hand start covers the rest, as the record says.

No findings. Every step's input is what the step before it produces, and every decision's end state meets its stated purpose.

Tool calls:
1. Grepped `launchStrandLocked` for pane binding and the `loom start --no-attach` spawn site: needed to check relaunch placement and who runs D5's healing start, which the manifest lacked.
2. Read `internal/battencli/wire.go:1255-1304`: the loom session spawn, to confirm it runs in the child's anchor.
3. Read `internal/reedengine/spawn.go:78-102`: when `s.PaneID` is rebound against when the old pane is killed, for relaunch placement.
4. Grepped `RecipePawl` outside shedrun: to confirm the batten pass seeds and spawns a pawl run, since the comment at `wire.go:1281` names only tapestry and darn.
