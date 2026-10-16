# reed: every strand gets its own window; retire pane-era placement

`lyx reed add` with no `--anchor` used to default to `below-parent`.
With no parent, the strand was split into the selvage `shell` window beside the Selvage terminal.
Every strand reed places now gets a tmux window of its own (or its group's), and the `shell` window holds only Selvage.
The pane-era placement that stacked strands in the shell window is removed from reed's vocabulary, layout code, config, bindings, help texts, docs and tests.
The #673 workaround is gone from the instructions (`ORCHESTRATOR.md`).

Closes #673

## What changes

- **Anchors**: `render.Anchor` is now `own-window`, `overview` and `hidden`; `AnchorBelowParent` and `AnchorTop` are removed.
- **CLIs**: `lyx reed add` and `lyx shuttle run` default `--anchor` to `own-window` and accept `own-window|hidden`.
  Any other value, a retired one included, exits non-zero with a message naming `own-window`, and adds or starts nothing.
  `--parent` keeps its lineage and recursive-remove meaning, and no longer implies placement.
- **Engine write path**: `AddStrand`, `AddStrandUnless`, `UpdateStrand` and `ReplaceStrand` refuse a display naming `below-parent` or `top`.
- **Old state**: a `reed.json` strand with a retired anchor loads and reads as `own-window`; the `size` and `contentRows` keys are no longer read.
- **Shell window**: render's stack policy (partition, ordering, height rules, band clamp, in-window focus), reed's split-target planning and `select-layout` on the shell window, `Display.Size`, `SetContentRows` and `ContentRows` are removed.
  Selvage is the shell window's only pane and fills it after boot, after a Selvage heal and after the migration below.
- **Config**: `collapsed_rows`, `min_full_rows` and the `selvage` block (`height_rows`) are `RetiredKeys` of reed's config spec and leave both templates, so an old `reed.yaml` loads and `lyx config reconcile` prunes them.
  `width` and `height` stay.
  The `watchdog` key stays and gates name repair; its comment is rewritten.
- **Resize self-heal**: the per-worktree watch loop's layout re-apply, the `window-resized` hook and its signal file are removed.
  Boot and every attach clear a `window-resized` hook on the shell window with the unconditional unpin (`set-hook -u`).
  The per-hub watchdog daemon keeps its per-session name-repair watch.
  The standalone reed-up callbacks in `burlercli` and `webstercli` lose their `watch` parameter.
- **Status bar and bindings**: line 1 drops the per-strand pane buttons and their click branch; Alt+z and the zoom-keeping wrapper are removed, and the bindings pin issues `unbind-key` for `M-z`.
- **Docs and tests**: package docs, `docs/overview.md`, the reed template comments, both reed sandbox suites and the tests are swept.
  Tests pinning retired behaviour are deleted, including the dot-fill smoke tests and the resize/zoom suites; tests that used `below-parent` as a convenient anchor use `own-window`.
  Sandbox scenarios that exercised only the retired layout or the self-heal are retired with a one-line reason.

## Design decisions

- **Migration of old panes**: the first step of every reconcile moves a bound strand pane found in the shell window into its own agent window with `break-pane`, keeping its process.
  It is placed before the shell window as a newly opened own window, and its record is rewritten with `anchor: own-window`.
  A dead pane is moved the same way and relaunches like any dead own-window pane.
  The step is standing, gated by no upgrade marker, so a strand an older or `.dev-bin` binary adds later is moved by the next op.
  It never touches Selvage or an unbound pane, and never breaks out the shell window's last pane (that would take the window); such a pane stays until an up or resume heals Selvage beside it, and the move after that heal takes it.
  In up and resume, the Selvage heal runs before the move.
- **Shell window identity**: with no recorded Selvage pane listed, the shell window is the window carrying the `@lyx_strands` marker, never the session's current window.
- **Retirement over remap in the CLI**: a refusal naming `own-window` was chosen over silently remapping, so a caller typing a retired value learns of it.
- **Out of scope**: window groups, `window_share_max`, `window_split`, window order and focus rules, the status bar's window buttons, line 2 and follow button, and Selvage itself are unchanged.
  `ownwindow.go`'s `select-layout` for a shared group window stays.

## To check by hand

- The `tmux` tier compiles with `go vet` but was not run, so the new real-tmux subtests (migration of a live and a dead strand pane, Selvage heal with the corpse as the only pane, `M-z` unbound after re-pin, `window-resized` unpin) have not executed.
  Run `lyx gate test --tags tmux ./internal/reedengine ./internal/reedcli` and the integration tier.
- Rewriting a retired `reed.json` through `ReedState` does not keep the old `size` and `contentRows` keys, because `ReedState` has its own decoder.
  The decision called for `internal/state` to keep unknown keys on a rewrite; check that dropping them is acceptable.
- `internal/reedengine/server.go:28` has a stale comment still mentioning the resize watch loop.
- Open finding: when no Selvage pane is listed, shell-window identity rests on the `@lyx_strands` marker (`d3-shell-window-identity-without-selvage`); check the move and heal in that state.
- Open finding: removing the stack layout breaks an `UpdateStrand` test (`card2-breaks-update-strand-test`); confirm the `UpdateStrand` tests in `internal/reedengine` still cover the retired-anchor refusal and pass.
- Open finding: `ORCHESTRATOR.md` also carried a #673 workaround (`orchestrator-md-673-workaround-survives`); confirm no mention of it remains.
- Open risk: a long-lived `lyx` on the old binary (an orch, a shuttle run) keeps the old stack layout and re-pins the `window-resized` hook until it ends; the next op on the new binary clears the hook, so the window settles once old processes are gone.
