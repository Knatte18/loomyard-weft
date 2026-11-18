```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing path that writes or reads the state D1–D3 touch.

**Paths traced**

D1, re-adding the loom status strand and writing its sidecar from `ensureRunsStrand`:
- **Next `lym loom start` / resume:** survives. `ensureStatusStrand` → `resolveStatusStrandAction` reads `statusSidecarPath(c.location)`, the same worktree's `LoomScratchDir`. The orch writes the build from `statuspane.CurrentBuild` (`internal/orchcli/cli.go:241`), the same identity the bootstrap compares, so the strand is kept.
- **Dead status strand later, then loom start:** survives. The bootstrap replaces it, and D2's builder picks beside-runs because the runs strand is present.
- **Second `lym beam start` with a current runs strand:** survives. The early return at `internal/orchcli/runs.go:82-83` leaves the status strand alone.
- **Stale runs strand:** survives. `ReplaceStrand` targets only the runs guid, and D1's re-add does not run.
- **`lym orch stop` / watcher stop:** survives. They remove the orch strand only (`internal/orchcli/stop.go:95`, `watch.go:129`), never the runs or status strand.
- **Reed resume or relaunch after a server restart:** survives. The strand keeps `Display.Window`, the `WindowGroups` record persists in `reed.json`, and `createOwnWindowPaneLocked` (`internal/reedengine/spawn.go:164-205`) reopens it under the same rule.
- **Reed's watchdog name repair:** survives. It repairs pane titles and provider session names, never window names (`internal/reedengine/namerepair.go:1`).
- **Shell-window break-out:** survives. `moveShellWindowStrandsLocked` clears `Display.Window` only for a pane left in the shell window, and an own-window add never lands there.
- **Reed's add path when the role is held:** breaks the record's mechanism (finding 2).

D2, the shared builder and recipe group:
- **Seed availability at bootstrap:** survives. `seedAndCommitBootstrap` writes or finds the seed before `ensureStatusStrand` runs (`internal/loomcli/start.go:524` then `:553`).
- **A task worktree with no runs strand:** survives. It gets the overview placement with no group, as today.

D3, the overview reading reed state for the window name:
- **Reed's window naming (`internal/reedengine/spawn.go:167,186,197`):** breaks the record's premise (finding 1).
- **`reedengine.LoadState` (`internal/reedengine/state.go:150`) → `state.ReadJSON` (`internal/state/state.go:96-107`):** breaks the record's "takes no lock" Constraint (finding 1).
- **Pane cwd resolution:** survives. The overview pane runs at `PaneCwd = l.AnchorPath()` (`internal/hubgeom/hubgeom.go:90-91`), so `lymcwd.Resolve` gives the beam's own worktree. Its reed state is under that anchor (`internal/reedengine/lifecycle.go:36`), and its seed and status are those of `shedrun.SelfRunID`.

**Findings**

1. MEDIUM · design · `decision-record.md` D3, and Constraints "The overview reads files only, takes no lock"
   - Reed's state does not hold the window name D3 says it reads. `Strand` persists only `Display.Window`, the group (`internal/reedengine/state.go:21-53`).
   - The window name is passed to tmux `new-window -n` at launch: the group when shared (`spawn.go:197`), else the role via `strandLabel` (`spawn.go:167,186`; `paneoptions.go:18-24`).
   - With `window_share_max` 0, `Display.Window` still reads the recipe while the window is named `loom-status`. Telling the two apart needs `WindowGroups[group].Shared` plus D2's rule (`ownwindow.go:204-224`).
   - So the Rationale's claim that D3 reads the name "instead of recomputing D2's rule" is false. Which reader and which state D3 uses is left to Plan; the alternative is a tmux query, which reads no file.
   - The obvious reader, `reedengine.LoadState`, takes a shared read lock (`internal/state/state.go:99`). That contradicts the Constraint that the overview takes no lock.
   - Fix: name the reader and the derivation (or the tmux query), and reconcile it with the no-lock Constraint.

2. MEDIUM · design · `decision-record.md` D1, the "add refused because a strand already holds the loom status role counts as done" sentence
   - Reed's add path gives no typed error for a held role. The refusal is a bare `fmt.Errorf` (`internal/reedengine/strand.go:226-227`), and the package's only sentinels are `ErrNoSession`, `ErrUnknownStrand`, `ErrNewWindowFailed` and `ErrWindowReadBack`. So D1 can tell this refusal apart only by matching the message text.
   - A dead holder is not refused at all: `reclaimDeadNameLocked` removes it first (`strand.go:246-251,663`).
   - An existing path already does what D1 wants without classifying an error: `AddStrandUnless(spec, unlessName)` skips the add, with no error, while a strand of that name exists (`strand.go:550-555`).
   - The record leaves Plan to work out how the refusal is detected.
   - Fix: D1 uses `AddStrandUnless` with `loomengine.LoomStatusStrandName`, or states the detection and its pin.

Tool calls:
1. `hubgeom.reedGeometry` and the head of `reedengine/namerepair.go`: needed to place the overview's reed state and check whether any path renames windows.
2. Grep for orchcli's runs-strand writers and `RemoveStrand`: needed to check whether stop or watch paths remove the runs or status strand.
3. Grep for orchcli's `currentBuild` wiring: needed to confirm D1's sidecar build matches the identity the loom bootstrap compares.
4. `internal/loomcli/start.go` 520-560: needed to confirm the seed exists before `ensureStatusStrand` builds D2's spec.
