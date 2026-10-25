```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering every function or branch the diff changes, with one concrete case run for each.

**Per-branch list**

- `atOrBelow` (`internal/battencli/overview.go:395-407`), chain a→b→a (or a→a) with the window's worktree c outside the cycle: **incorrect**. The loop has no visited set, so it never terminates (finding 1).
- `atOrBelow`, slug equal to the window's worktree: correct. It returns true before calling `parentOf`, so a beam's own orch always shows even when its parent lookup would fail.
- `atOrBelow`, slug a direct child of the prime: correct. `ParentOf` returns "", the loop ends, and the result is false. The prime's orch and sibling beams are excluded.
- `atOrBelow`, slug whose parent worktree no longer exists: correct. `parentOf` on the missing parent either returns "" via the board and ends the loop, or errors and the orch is left out, logged at Debug.
- `orchsAtOrBelow`, prime window: correct. It returns `locations` unchanged.
- `orchsAtOrBelow`, beam window: ordering is not as documented (finding 2).
- `windowOrchPaths`, `HubOrchs` error: correct. It returns the error, and `orchLines` falls back to the window's own orch.
- `windowOrchPaths`, labels: correct. The prime gets "", and a beam gets its slug via `location.WorktreeName`.
- `orchLines`, `orchs` nil or failing: correct. The own-orch fallback uses `orchLabel`.
- `orchLines`, empty list (own orch not listed): correct. It draws no orch lines and the frame is dobby-only.
- `frame()`, dobby nil with orch lines: correct. The orch lines show alone with no leading blank line.
- `frame()`, dobby lines with orch empty: correct. There is no trailing blank line.
- `frame()`, both present: correct. The order is dobby lines, "", then orch lines.
- `overviewRunCells`, batten not found: correct. It shows `starting`, stage "-", with the time falling back to `StartedAt`.
- `overviewRunCells`, batten unreadable, or in Run-Shed with the child unreadable: correct. It returns `Unreadable`.
- `overviewRunCells`, Run-Shed with the child not found: correct. It keeps batten's own Run-Shed stage.
- `overviewRunCells`, live pause on a running, paused or starting run: correct. It shows `paused <minutes>`.
- `overviewRunCells`, Run-Shed with the child found: correct. It uses the numbered child stage (`6.6 Plan-Write`) and the child's last-entry time.
- `DobbyRunCells.Of`, pause record unreadable: correct. It is logged at Debug and covers nothing.
- `DobbyRunCells.Of`, `view.Runs` empty: correct. It returns an empty map.
- `DobbyRunCells.poll`, seed absent or unreadable: correct. Nothing is cached and the next frame retries. An absent child seed is re-read every frame, which costs a read but gives no wrong result.
- `DobbyRunCells.poll`, `childStatus` error: correct. `Child` stays the zero poll and the batten stage is shown.
- `DobbyLines`, a slug missing from the cells map: not wrong with the two current callers. The zero `RunCells` draws blank padded cells rather than `status unreadable`. Both callers pass `cells.Of(view.Runs, ...)`, so every slug is present.
- `DobbyLines`, all runs unreadable: correct. The widths stay 0 and the lines show `status unreadable`.
- `beamcli.dobbyStatus`: correct. It builds one `DobbyRunCells` for all worktrees and calls it sequentially. A `PauseGeometry` failure is a new failure path that now fails the whole status; `overviewCmd` already required it before the diff, so this is consistent.
- `overviewCmd`, `worktreeHasDobby` error: correct. It warns and draws the orch section alone.
- `runsWindowTitleOf`: correct. A beam gets `beam: <slug>`, anything else gets `beam: prime`.
- `overviewWindowTitle`: correct. An empty `Title` gives `overview`, and a task pair's loom status strand keeps its name.
- `openOwnWindowLocked`: the title reaches `new-window -n` (spawn.go:288) as a single argv element, and the final `rename-window` (per `reedengine/doc.go:344`) also takes it as one argument. The title is not parsed as a tmux target anywhere. The only `window_name` readers I found are the `#{window_name}` format strings and the tier2 test comparisons; no code looks a window up by name. Correct.

**Findings**

1. MEDIUM, `design`, `internal/battencli/overview.go:395-407` (`atOrBelow`, called from `orchsAtOrBelow` at :382).
   - The parent-chain walk has no cycle guard. A self-parent or a two-beam cycle, in a malformed origin record or board parent, loops forever for every beam window whose worktree is outside the cycle. `DobbyParents.Of` returns the cached value instantly (`dobbyreads.go:47-58`), so the pane's frame function spins at full CPU and the pane never redraws.
   - The prime window is unaffected because `orchsAtOrBelow` returns early.
   - The focus directive asks whether this terminates on a cyclic chain, and it does not. The fix is a visited set, or a bound on the number of steps, that returns an error or false on revisit.

2. LOW, `design`, `internal/battencli/overview.go:354-371` and the comments at :326 and :409.
   - The comments say a beam window's orch list has "its own first" (`orchs` field comment at :326, `orchLines` comment at :409). `windowOrchPaths` keeps `HubOrchs` order, which is the prime first and then git worktree list order (`internal/orchcli/live.go:28-45`). A beam's own orch is therefore not guaranteed to come first when nested beams precede it in that list. Only the order of the lines is wrong. Either reorder the shown list so the window's own comes first, or drop the claim from the comments.

**Tool calls**

1. `grep` for `rename-window` and `-n` in `internal/reedengine/*.go`, plus `HubOrchs` in `internal/orchcli/live.go`: to confirm the window title is never parsed as a tmux target, and to learn the order `HubOrchs` returns, which the manifest lacked.
