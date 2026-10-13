# reed: overview window first, agents in start order, shell last

A reed session's windows now follow one fixed order: the run overview first, then one window per agent in start order, then the shell (Selvage) last.
Attach and every automatic window switch show the most relevant running agent, fall back to the overview, and never land on the shell.
Before this, window 0 held Selvage, the LLM driver and the status pane above them, and `lyx reed attach` landed there.

## What changes

- A new `render.AnchorOverview` (`"overview"`) places one strand in a window of its own, opened first.
  A second `overview` strand under another name is refused, naming the existing one.
  `lyx loom checklist` (loom and darn runs) and `lyx batten overview` use it.
- The loom LLM driver strand and the orch's orchestrator strand change from `below-parent` to `own-window`.
  In the prime the operator now sees the batten overview or the orchestrator, no longer both at once.
- Own-window strands open before the shell window, so window index order is start order.
  `OpenWindow`'s hidden windows open after the shell.
  Boot renames Selvage's window `shell`.
  Where psmux refuses `-b` or `-a`, placement falls back to the plain form, so the order is best-effort on Windows.
- Focus:
  - Attach selects the newest live agent's window, else the overview, and leaves the window alone when another client is attached or the client listing fails.
  - A new agent window is selected only when the current window is the overview or an agent window whose panes are all dead; otherwise it keeps `@lyx_unseen`. A `Focus: true` strand is always selected.
  - When the viewed window ends, reed selects the newest remaining live agent's window, else the overview, never the shell.
- Alt+Up and Alt+Down step across visible windows and wrap, skipping hidden windows, in every window.
  Stepping between panes of the shell window is dropped.
  The `shell` button replaces VIEW with the same action.
- The loom checklist frame shows a detail block per visited row (each visit's time, duration, outcome, output and gate attempts, read from the existing shed status history), then the full row list, then the state line and the full wait reason.
  An escalation wait reads "waiting on the parent: escalation from …".
- Both overview panes no longer report content size or truncate; reed sizes nothing in a window of their own and every row is drawn.
- New leaf package `internal/selfexec`: when the binary at the recorded path changes identity on two consecutive polls, the process `syscall.Exec`s itself with the same argv and environment, keeping its pane and strand binding.
  `statuspane.Pane.Run` calls it on each poll tick.
  It never execs under `go test` and does nothing on Windows.
- `reapplyLayout` resolves the live box first and skips the apply, and the zoom toggle, when the box equals the last applied box or the box whose apply last failed.
  This removes the flicker on a re-apply that changes nothing.
- A live session booted by an older `lyx` is brought into order on the next window open or attach: its strands' window is renamed `shell` and moved after the last visible window.
  Already placed strands stay put, so a resumed `below-parent` driver stays in the shell window until its run ends.

## Design notes

- The selection on attach runs under the op lock ahead of the layout chain's guards, so it holds with Selvage alone in the shell window and with no client size known.
- Focus and navigation read reed state, window ids and pane liveness only, never `@lyx_unseen`, `@lyx_hidden` or wait marks (`PATTERN-wait-mark-display-only`).
- "Newest live agent" stands in for the current row's agent, since a run spawns its current row's agent last.
- `AnchorTop`, `AnchorBelowParent`, content sizing and the stacked layout stay valid for explicit callers and keep working in the shell window.
- `Engine.SetContentRows` and the `InstallPins` apply option stay, with no overview-pane caller.
- Restarting other long-lived verbs on a new binary is left to the board entry `stale-binary-daemons`, which can call the `internal/selfexec` helper.
- `pattern/PATTERN-leaf-packages.md` gains an `internal/selfexec` section, pinned by `leaf_enforcement_test.go`.
  Docs are updated in `internal/reedengine`, `internal/statuspane`, `internal/selfexec`, `internal/loomcli`, `internal/orchengine` and `internal/shuttleengine`, and in `docs/overview.md`.

## Check by hand

- The `tmux` and `integration` test tiers of `internal/reedengine` were compiled but not run during the work; run them, including `windoworder_integration_test.go`.
- Open finding: the restored `SetContentRows` doc comment and `doc.go` paragraph in `internal/reedengine` carry retry clauses that describe a pane reporter this change deletes (`apply.go`, `doc.go`); reword or drop them.
- Open finding: `attach-select-moves-other-attached-clients`. Confirm in `internal/reedengine/attach.go` that an attach with another client attached, or a failed client listing, never moves that client's window.
- A skip at the box whose apply last failed still logs the "recovered" Info line in `reapplyLayout`; confirm the wording is acceptable.
- A new build that exits at once on the same argv leaves the overview pane dead until `lyx loom start` or `lyx orch start` replaces it.
- A re-exec'd overview pane keeps a sidecar naming its starting build, so the next `lyx loom start` or `lyx orch start` replaces it once more.
- tmux copy mode shows a frozen frame, so a redraw appears only after leaving copy mode; a row list taller than the window scrolls its earliest rows off the top.
- Attach an old live session and a fresh run in a real terminal, and step with Alt+Up and Alt+Down, to see the order, focus and `shell` button on tmux, and the best-effort order on psmux.
