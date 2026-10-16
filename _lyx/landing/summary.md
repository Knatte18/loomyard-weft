# reed: give every strand its own window, retire pane-era placement

`lyx reed add` with no `--anchor` used to default to `below-parent`, which split a parentless strand into the selvage `shell` window beside the Selvage terminal.
That default is left over from when strands were panes in one window.
Every strand reed places now gets a tmux window of its own (or its group's), and the `shell` window holds only Selvage.

Closes #673

## What changes

- `render.Anchor` is now `own-window`, `overview` and `hidden`; `AnchorBelowParent` and `AnchorTop` are gone.
- `lyx reed add` and `lyx shuttle run` default `--anchor` to `own-window` and accept `own-window|hidden`.
  Any other value, a retired one included, is refused with a message naming `own-window`, and adds nothing.
  `--parent` keeps its lineage and recursive-remove meaning.
- The engine's display check on `AddStrand`, `AddStrandUnless`, `UpdateStrand` and `ReplaceStrand` refuses a retired anchor.
  A `reed.json` strand carrying one still loads and reads as `own-window`.
- Reconcile starts with a standing step that moves a bound strand pane found in the shell window into its own agent window with `break-pane`, keeping its process, and rewrites its record to `own-window`.
  A dead pane is moved the same way and relaunches.
  The step never touches Selvage or an unbound pane, and never breaks out the shell window's only pane; such a pane stays until an up or resume heals Selvage beside it.
- The shell window's stacked layout is removed: render's stack policy and height rules, `Display.Size`, `SetContentRows`, `ContentRows`, split-target planning, `select-layout` on the shell window and the pane-border pin.
  Selvage is the window's only pane and fills it.
- The resize self-heal is removed: the per-worktree watch loop's layout re-apply, the `window-resized` hook and its signal file.
  Boot and attach unconditionally unpin `window-resized` on the shell window, which clears a hook an older binary set.
  The per-hub watchdog keeps its per-session watch as name repair alone, still gated by `watchdog`.
- The per-strand pane buttons on status line 1, the strand click, Alt+z and the zoom-keeping wrapper are removed.
  The pin issues `unbind-key` for `M-z`.
- `collapsed_rows`, `min_full_rows` and the `selvage` block (`height_rows`) are `RetiredKeys` in reed's config spec, so an old `reed.yaml` loads and `lyx config reconcile` prunes them.
  `width` and `height` stay.
- Package docs, `docs/overview.md`, config template comments and both reed sandbox suites are swept.
  Sandbox scenarios that exercised only the retired layout or the resize self-heal are retired with a one-line reason.
  The #673 workaround is removed from `CLAUDE.md`.

## Design decisions

- `below-parent` is removed from the vocabulary rather than just defaulted away, because a surviving flag value would still put a strand in the shell window.
- The pane move is a standing reconcile step with no upgrade marker, so a strand that an older or `.dev-bin` binary adds to the shell window later is moved by the next op.
  Moving it keeps the operator's live session and lets every own-window rule apply unchanged.
- The retired config keys use configengine's `RetiredKeys`, as `collapsed_strip_rows` did.

## To check by hand

- The `tmux` tier compiled with `go vet` but was not run, so the new real-tmux subtests have not executed.
- Rewriting a retired `reed.json` does not keep its old `size` and `contentRows` keys, because `ReedState` has its own decoder.
  The design intended `internal/state` to keep them as unknown keys; confirm the loss is acceptable or fix it.
- Plan finding, scope: the removal of the stack layout breaks an existing `UpdateStrand` test; check that the test was updated to the new behavior rather than weakened.
- `internal/reedengine/server.go:28` has a stale comment that still mentions the resize watch loop.
- A long-lived `lyx` process on the old binary (an orch, a shuttle run) keeps the old stack layout and re-pins the `window-resized` hook until it ends; the next op on the new binary clears it.
- `internal/shuttlecli/cli_test.go`: two subtests were renamed to satisfy a grep over retired anchor names; check the names still describe what they test.
- Several decisions (anchor refusal, migration, resize retirement, button removal) were taken without the operator in auto mode.
