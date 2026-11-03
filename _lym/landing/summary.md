# Show a beam's pawl checklist in a second overview window

A beam's session now shows the pawl run's checklist in an overview window named `pawl`, right after the beam overview.
Before, `lym beam start` made the runs strand replace the loom status strand in the session's single overview window, so the checklist disappeared and the operator could not see where pawl stood.

Neither strand replaces the other, whichever of `lym loom start` and `lym beam start` adds its strand first.

## Design decisions

- **reed overview anchor holds several strands.**
  `render.Display` gains an integer `Order`, read only for an overview strand and persisted in `reed.json`; its zero value is the first place, so displays written by an older lym load as key zero.
  Overview windows sit at the lowest indexes in ascending `Order`.
  A new, relaunched or replacement overview window opens after the last overview window whose key is not higher than its own, so equal keys sit in the order they last opened.
  `AddStrand` no longer refuses a second overview strand, and `ReplaceStrand` no longer refuses while a third strand holds the overview.
  `UpdateStrand` refuses a change to an overview strand's `Order`, as it refuses an anchor move, and names remove-and-add as the way forward.
  reed learns no beam, loom, pawl or recipe name; each spawner supplies its key and title.
- **Focus fallback takes the lowest-index overview window.**
  `relevantWindow` (through `agentWindowPanes`) and the engine-less `followTargetWindow` behind `lym reed follow` pick the lowest-index overview window, so in a beam's session the fallback still leads to the beam overview.
  A live agent window still wins over every overview window.
- **Loom status strand always takes an overview window.**
  `statusStrandAddSpec` now takes the run's recipe name and returns an overview display with `Order` one past `orchcli.RunsStrandOrder` and the recipe as its title.
  The branch that gave it an own window with `KeepViewer` beside a runs strand is gone.
  The title comes from the run's recipe, not from a branch on a recipe name, so a pawl run reads `pawl`, and a task run's window, formerly `overview`, now reads `tapestry`, `darn` or whichever recipe it runs.
  The window is named with the recipe alone, not `pawl: <slug>`, because the beam overview already names the slug and a short name keeps status line 1 narrow on a phone.
- **Runs strand is added beside the status strand.**
  `ensureRunsStrand` no longer looks for a loom status strand and no longer replaces one.
- **No migration step.**
  A session from before this change heals at the pawl run's next `lym loom start`, which replaces or adds the status strand into the second overview window.
  Until then it keeps the old placement.
  If the old status strand opened first, the focus fallback lands on its window rather than the beam overview, because the old key ties with the runs strand at zero.
  A pawl driver started by the old build runs the old `ensureRunsStrand` in-process at its Beam-Up row and can drop the runs strand or replace the status strand; a later `lym beam start` or `lym loom start` under the new build restores them.
- Package docs in reed, orchcli, loomcli and orchengine, and `docs/overview.md`, are updated to drop the one-overview-strand rule.

Out of scope: a line in the beam overview naming pawl's current row, a split of the overview window into panes, and any change to the order in which `lym beam create`, `lym beam start` and the pawl run's `lym loom start` run.

## To check by hand

- The tier2 tests have not run, so the real-tmux tests added or changed here are unverified; run them, notably `internal/reedengine/windoworder_integration_test.go` and `internal/loomcli/smoke_drivingsurface_test.go`.
- Package docs that describe the orch entry's overview window may still state one overview strand per session (finding `orch-entry-overview-doc-stale`, consistency, medium): grep the reed, orchcli, orchengine and loomcli docs and comments for it.
- In a real beam session, confirm the window order is beam overview, `pawl`, then agents, shell and hidden windows, under both start orders.
