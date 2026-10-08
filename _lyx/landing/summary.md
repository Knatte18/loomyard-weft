# reed: two-line bar, zoomable strands, segment colors, run switching

A reed run is now readable on a small screen.
Each strand can be viewed alone as a zoomed tab with an overview to return to, every run is a tab that scrolls instead of falling off a narrow terminal, and each strand role has its own color in tmux and in Claude Code.

## What changes

- **Two-line tmux bar.**
  Line 1 holds `VIEW`, one button per strand (`@strand`) and, when the session has several windows, one button per other window.
  Line 2 holds one button per session in session-id order, the current one by its full name and the others cut to 12 characters, and the clock.
  Both lines use list mode, so an overflowing line scrolls and keeps the lit button visible.
  A strand with a wait mark shows `⏳<label> <elapsed>m` on its button.
  Strand panes carry a border title with their label.
- **Zoom as the strand view.**
  `VIEW` is the overview (reed's existing layout); a strand button zooms that pane.
  Every reed step that changes the strand window's panes, layout or active pane, the resize hook and the attach chain keep a zoom across the step, since tmux unzooms on `resize-pane -y`, `select-layout`, `split-window`, `kill-pane` and `select-pane` without `-Z`.
- **Keys and clicks.**
  Alt+z toggles zoom, Alt+Up/Down step through strands keeping the zoom, Alt+Left/Right switch run.
  `MouseDown1Status` is replaced by a dispatcher for the four button kinds; other mouse bindings keep tmux's defaults.
  Pins are issued at boot and again in the attach pre-flight, so a server booted by an older lyx picks them up at the next attach.
- **`lyx reed switch --next|--prev`.**
  Steps a client through the server's sessions in session-id order, wrapping.
  It takes every input as a flag (`--socket`, `--client`, `--tmux`), resolves nothing from a working directory, and prints nothing on success.
- **Segment colors.**
  New leaf package `internal/segmentcolor` declares the loom segments and a closed palette.
  Reed's config gains one color key per segment; the template defaults are blue (coordinator), purple (discussion), cyan (plan), green (webster), orange (review), yellow (describe) and pink (landing).
  Each spawning module names its segment on its launch spec.
  Reed sets `@strand_color` for the button and border title, and shuttle types the provider's color command (`/color <name>` through `claudeengine`) before the skill load.
  The orch watcher types the orch's color at start and first in every reload.
- **Spawn order across a restart.**
  When a worktree's recorded session is revived, reed first revives the earlier worktrees' sessions in spawn order (prime, then pairs by run start), so session ids keep the order the bar and `switch` use.
- **VS Code keys.**
  `lyx ide spawn` writes a `// lyx:begin` … `// lyx:end` block of `sendSequence` entries into the user's `keybindings.json` so VS Code forwards the Alt keys.
  A skipped seed (unreadable, unwritable, no array, malformed markers) is reported and never fails the spawn.
- **Retired.**
  The single-line status pipeline is gone: the `status-line.md` template, the `status_line` config key, `lyx reed statusline`, the `waits` token and `internal/tokenvocab`.
  An existing `reed.yaml` that still carries `status_line` loads with the key ignored.

## Design decisions to review

- **Geometry.**
  `pane-border-status top` draws the title inside the top cell, so the pin on the pane at window row 0 is its cell height minus one row and every other pin keeps its cell height.
  The adjustment is keyed on a readback of exactly `top`, and a planned one-row top cell is floored at one content row, ending one row taller than planned.
  Pins also run right after every layout apply and at the end of the attach chain, not only from the resize hook.
- **Pane order is read unzoomed.**
  A zoomed pane reports `pane_top` 0, so every pane list a layout or pin is planned from is read on the unzoomed window; the hook and attach chain test `#{window_zoomed_flag}` when they run, not when reed builds them.
- **Pins are global or scoped deliberately.**
  `status-format[0]`/`[1]` are global only, because a per-session index would drop the global lines for that session; `status` is session-scoped; border options are scoped to the strand window, marked by the `@lyx_strands` window option so `batten:` windows are never read as strands.
- **Key bindings take five keys.**
  Alt+z, Alt+Up/Down and Alt+Left/Right no longer reach programs in a pane, Claude Code included.
  The `M-Left`/`M-Right` binding names the absolute `lyx` path from `panebin.go`'s chokepoint, shell-quoted through `internal/shell` with `#` doubled for `run-shell`'s format expansion; `PATTERN-pane-binary-resolution` widens to cover it.
- **Color validation.**
  `LoadConfig` validates nothing; a color outside the palette is refused only in `ensureServerAndSessionLocked`, so `down`, `status`, teardown and attach to a live session keep working and show no color for that segment.
- **Color is display only.**
  A failed color step is logged and never fails a launch.
- **Spawn-order revival** is sequenced by `reedengine` from a sorted list the hub-mode teller supplies; `reedengine` derives no path, holds no two op locks at once, and a failed predecessor is logged and skipped.
- **Docs.**
  `PATTERN.md` and `pattern/` gain the color rule and the widened `PATTERN-pane-binary-resolution`, `PATTERN-leaf-packages` adds `segmentcolor` and drops `tokenvocab`, and the touched packages' `doc.go` files are updated.

## To check by hand

- Run the `tmux`-tier tests (`internal/reedengine`, `internal/reedcli`); the plan's verification does not run that tier.
  `TestSmokeDotFill/CrossClientControl` failed earlier because its 8-row pane no longer shows the first strand's marker under the two-line bar; a later change to its expectation should be confirmed green.
- Look at a live session on tmux 3.6: the bar on a narrow terminal, a zoomed non-top strand across a window resize, re-attach, add and resume, and the Alt keys and clicks.
- Whether Claude Code keeps a `/color` choice across `--resume` is unverified; a strand reed resumes may show the default prompt-bar color while its tmux tint stays correct.
- A strand recorded before this change gets `@strand` at the next attach but no `@strand_color` or `/color` until it is respawned.
- On WSL, VS Code reads the Windows-side `keybindings.json`, which `lyx ide spawn` inside WSL does not write.
- `docs/benchmarks/test-redundancy.md` is a generated report that still names deleted tests; regenerate it.
- Windows (psmux): the new pins and bindings are attempted unbranched and may be refused, as the old status line already was.
