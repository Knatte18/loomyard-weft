# reed: overview window first, agents in start order, shell last

A reed session's windows now follow one fixed order: the run overview, then one window per agent in start order, then the shell (Selvage) last.
Before, window order was tmux creation order, `lyx reed attach` landed on Selvage, and a new agent window opened detached and unseen, so the operator had to hunt for the agent at work.

## What changes

- A new `overview` anchor (`render.AnchorOverview`) places one strand in the first window.
  `lyx loom checklist` (loom and darn runs) and `lyx batten overview` move onto it; a second `overview` strand under another name is refused with a message naming the existing one.
- The loom LLM driver and the orch's orchestrator each get their own window.
- Selvage's window is named `shell` and kept after the last non-hidden window; its status-bar button replaces VIEW, and clicking it selects and unzooms the window.
  A live session booted by an older `lyx` is brought into the order on the next attach or window open.
- Focus: attach selects the newest live agent's window, else the overview, else nothing, and never the shell.
  A new agent window is selected only when the current window is the overview or a dead agent window; otherwise it carries `@lyx_unseen`, and a `Focus: true` strand is always selected.
  When the viewed window ends, the newest remaining live agent's window is selected, else the overview.
- Alt+Up and Alt+Down step across visible windows in index order and wrap, skipping hidden `OpenWindow` windows.
- The checklist frame shows the full row list, the wait reason in full (an escalation worded as waiting on the parent) and a detail block per visited row.
  Both overview panes no longer report content size or truncate.
- New leaf package `internal/selfexec`: `lyx loom checklist` and `lyx batten overview` re-exec themselves once a changed binary identity holds over two checks, so a deploy reaches runs already in flight.
  A failed stat never execs, a failed exec keeps the process running and is not retried for that identity, and nothing execs under `go test`.
- `reapplyLayout` issues no `resize-pane -Z` when the box is the last applied one or the one whose apply last failed.
  This removes the zoom flicker the hub watchdog caused every poll cycle.

## Design decisions

- `AnchorTop`, `AnchorBelowParent`, content sizing and the stacked layout stay valid for explicit callers and keep working in the shell window.
- The `applyOpts.SkipWhenBoxEquals` option and `SetContentRows` were removed as dead once the panes stopped sizing themselves.
- The watchdog's restart on a new binary is not part of this change; it is retired by `logging-cleanup`.
  Restarting `lyx orch watch`, `lyx reed attach` and other long-lived verbs is left to the `stale-binary-daemons` board entry, which builds on `selfexec`.
- Docs moved with the code: `internal/reedengine/doc.go`, `internal/statuspane/doc.go`, `internal/loomcli/doc.go`, `docs/overview.md` and `pattern/PATTERN-leaf-packages.md`.

## To check by hand

- The `tmux` test tier was compiled but not run; run `lyx gate test --tags tmux ./internal/reedengine/...` or the sandbox reed suite.
- Finding, unresolved by review: `shell-window-of-outside-selvage-seam` (consistency, blocking) in the code that finds the shell window when the recorded Selvage pane is outside the strands' window.
  Check that the `shell` rename and move behave there, and that with no recorded Selvage pane an attach or window open renames and moves no window.
- Open question: an attach while another client is attached must leave the current window unchanged; `attach-select-moves-other-attached-clients` was disputed in review, so confirm with two attached clients.
- Look at a live run by eye: window indexes, the `shell` button, Alt+Up/Down wrap, and the checklist at a real terminal size.
- `TestWatchLoop_NonSentinelFailureDoesNotGoDormant` was changed to fit the new skip rule; a skip at the failed box still logs the "recovered" Info line.
