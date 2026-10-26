# A read creates directories: state.ReadJSON mkdirs, so a hub reader leaves a real _lyx in a half-built pair

`hub-reader-creates-lyx` · Note · bug, fabric, orch, batten

## Seen (scripted-edits-allowed, 2026-10-26)

Worktree-Create stopped with "warp repo already contains a real _lyx … it predates weft" on a brand-new task worktree.
The shed's tackler sent it back to Worktree-Create without finding the cause, and the second attempt passed by luck, about 40 seconds later.

## Cause

A hub-wide reader wrote into the half-built pair.
The `lyx batten overview` window refreshes through `orchcli.HubOrchs` (`internal/battencli/overview.go`), which walks every git worktree, a pair still being created included, and calls `fabricengine.ReadOrigin` (`internal/orchcli/live.go`).
`ReadOrigin` reads `<warp>/_lyx/fabric/origin.json` through `state.ReadJSON`, which runs `os.MkdirAll` on the file's directory (`internal/state/state.go`), so the read creates a real `<warp>/_lyx/fabric`.
In `Topology.Add` the weft worktree exists about 150 ms before `WireJunctionsWith` runs, and a poll in that gap leaves a real `_lyx` the junction check refuses.
The tender's `hubOrchs` (`internal/tendercli/deps.go`) takes the same path, and a read during staging also leaves a stray `.weft` directory under `.fabric-wt-staging-*`.
It can recur on any `lyx fabric add` while an overview window or the tender runs.

## Idea

- A read never creates a directory, anywhere: `state.ReadJSON` drops its `os.MkdirAll`, so only writes create directories, and a missing directory reads as a missing file (operator, 2026-10-26).
  Every caller that leaned on the read's mkdir, such as one whose lock file sits in the same directory, is found and fixed in the same change, and `ReadJSONNoMkdir` folds into `ReadJSON`.
- `ReadOrigin` and every other reader that seeds an exclude or a lock directory before reading stop doing so for a read.
- `HubOrchs` skips a worktree whose `_lyx` is not yet a link and every path under `.fabric-wt-staging-*`.
- `Topology.Add` wires the junctions right after the weft worktree is created, before writing the launchers.
- A test holds `Add` between weft creation and wiring, runs `HubOrchs`, and asserts no `<warp>/_lyx` appears.
