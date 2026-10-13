# Fixer report — plan, round 2

## Changes

- **F1** (fixed as proposed): card 5 lists `internal/reedengine#windowCreationAllowlist` under `**Edit:**`, and its Intent says the allowlist drops the `window.go` entry, whose `new-window` literal moves into `newWindowLocked`, and that the `spawn.go` entry's `Why` names the one window-creation helper both own windows and `OpenWindow` open through.
  Premise confirmed: `window.go:82` holds the file's only `"new-window"` literal and `TestPaneCreationSitesRouteThroughThePreludeChokepoint` calls `RequireNoStale`.
  `lyx loom validate-plan` passes with informational findings only; the written files were checked directly, so `--overlay`, which takes proposed content on stdin, was not needed.
- **F2** (fixed as proposed): card 5 rewrites the Detached windows argv sentence (`doc.go:306`) with the `-a -t <highest-index window>` placement and its plain-form fallback.
  Card 7 rewrites the Own windows "never takes focus" sentence (`doc.go:317`) and the removal sentence (`doc.go:328`) to the relevant window, never the shell.
  Card 8 rewrites the geometry-pins bullet's `status-format[0]` button list (`doc.go:862`) to the strand, window and `shell` buttons.
- **F3** (fixed as proposed): card 5's `TestWindowOrder` keeps only the no-shell and no-window rows and says it holds the fallback cases the real-tmux rows cannot reach.
  Card 6's `overviewWindowInsertArgs` rows keep the hidden-lowest-window and no-window rows and drop the plain lowest-window row.
- **F4** (fixed as proposed): card 5's `OpenWindow` sentence says a nil state records no Selvage pane, so the shell check does nothing and logs at Debug.
  Its `TestOpenWindow` sentence says every existing row's exact `wantArgv` is rebuilt as `new-window -d`, the placement, then the caller's arguments.
- **F5** (fixed as proposed): card 6's `validateDisplay` subtest gains the case of an overview display with `Window` empty and `WindowSize` non-zero, wanting the window-group refusal.

## Disputed

None.

## Deferred

None.
