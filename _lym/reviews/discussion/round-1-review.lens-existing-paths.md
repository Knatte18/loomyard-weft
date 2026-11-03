```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state the record introduces or changes, trace the code paths that already write, reapply or read it, and say whether the decision survives each one.

**Paths traced, per decision**

- **D1, the order key stored in the persisted `render.Display`:**
  - `updateStrandLocked`/`refuseWindowMove`: survives. D1 adds the order-key refusal there (strand.go:441-470, 87-95).
  - `moveShellWindowStrandsLocked` rewrites `Anchor` to own-window: survives. It acts only on strand panes found in the shell window, and an overview pane always has its own window (ownwindow.go:392-422).
  - `AddStrandUnless`'s `ifAbsentRelaunch`: survives. It relaunches without rewriting the display (strand.go:624-658).
  - `ReplaceStrand`'s `moveStrandTo`: survives. Table slot order does not decide placement when keys differ (replace.go:88).
  - shuttle `run.go`/`attach.go` display passthrough: survives. No shuttle spec sets the overview anchor, and `lym shuttle run --anchor` refuses overview (shuttlecli/run.go:113).
- **D1, overview placement at open, relaunch and replace:**
  - `openOwnWindowLocked` already has reed state and `windowByPane`, so it can read each window's key from reed state's displays (spawn.go:261-283): survives.
  - `Resume` relaunches dead strands in strand-table order, one `launchStrandLocked` each (lifecycle.go ~862-870). With distinct keys each opening follows the key rule. With tied keys the table order becomes the "last opened" order. Survives.
  - `ensureShellWindowLocked`/`planShellWindowFix` moves only the shell window, behind the highest non-hidden window. Overview windows stay at the front (windoworder.go:160-230): survives.
  - Status bar line 1 buttons follow window index order and render `#{window_name}` (bar.go:36-47): survives, and the bar reads beam overview, then `pawl`.
- **D1, cap removal:**
  - The refusal in `addStrandLocked` is the only cap. `ReplaceStrand` reaches it through `addStrandLocked` (strand.go:404-410, replace.go:81), so removing it there clears both refusals: survives.
  - `testkit/shuttlefake` never modelled the cap: survives.
- **D2, focus readers:**
  - `relevantWindow` through `agentWindowPanes` (single `overview` string) feeds three callers: `selectRelevantWindowLocked` (FollowRun, ownwindow.go:526), the viewed-window-ending path from `RemoveStrand`/reconcile (strand.go:856, ownwindow.go:521), and `attachWindowSelectionLocked` (ownwindow.go:563). All are covered by D2's rewrite: survives.
  - `followTargetWindow`/`followSessionVia` read the `@lym_overview` marker, set only in `markOwnWindowLocked` (spawn.go:320, follow.go:62-85). Covered by D2: survives.
  - `ReplaceStrand`'s overview branch selects the new window when the viewed one ended (replace.go:97-99): survives, and is kept by D1's criterion.
- **D3, status strand display and window name:**
  - No code looks up a window by the name `overview`. `overviewWindowName` is only the default title (windoworder.go:20-27), and the bar and listings only display the name. Renaming the task session's window to its recipe name: survives.
  - `ensureStatusStrand`'s replace and add both use `statusStrandAddSpec`, and only `start` calls it (start.go:553): survives.
  - `KeepViewer` was the only guard against the own-window status strand taking the viewer. An overview window is never selected on open (spawn.go:281-286), and the record's Constraints state this: survives.
- **D4, `ensureRunsStrand`:**
  - Keep/replace through the runs sidecar and `StrandCurrent` (runs.go:82-96, build.go:80): survives.
  - Its only caller is `orchcli` start (start.go:483): survives.
- **D5, healing:**
  - `resolveStatusStrandAction` replaces a live strand whose sidecar build differs (bootstrap.go:226-236, build.go:80). An upgraded lym is a new build, so the first `lym loom start` replaces the strand. A session where the runs strand replaced the status strand takes the add path. Survives.
  - `lym loom step` and `lym loom resume` never call `ensureStatusStrand` (only start.go:553 does), so they do not heal. D5's bound limits the old placement to "until that start", so the record is not silent on this: survives.

**Findings**

None. No existing path undoes or misreads a decision without the record saying so.

**Tool calls**

1. Grep for `isOverview`, `@lym_overview` readers, and shuttlefake's `AddStrand`: to find every reader of the overview anchor and marker beyond the manifest.
2. Grep for window-name lookups (`overviewWindowName`, `move-window`, `window_name`) and the Up/Resume/reconcile definitions: to check whether D3's rename or D1's placement meets a lookup by name or a reorder path.
3. Grep of `Resume`'s relaunch loop in lifecycle.go: to confirm that relaunch order on a server rebirth follows strand-table order.
