# run-window-bugs: shuttle launch line, status pane, loom fmt, worktree leftover retry

Both batches finished `done`; all seven cards are committed.

## Batch 01 (cards 1–5), head `a1b424dfc`

- Shuttle's skill-load message rides the launch line, and the run awaits its turn before the color, retry and prompt.
- `start`'s startup probe reads a session start then a turn start from the events before the pane.
- The status pane clears screen and history on every redraw and wakes on a resize.
- `PushWithRetry`'s doc comment names the weft main ref-update race.
- The datagram stale-socket helper moved into `tmuxkit` and the dead-socket test uses it.

Deviations from the declared file-ops:

- `internal/shuttleengine/claudeengine/command.go`: one stale doc-comment line on `buildLaunchCmd` updated.
- `cmd/lyx/tierpurity_test.go`: a new `allowedSpawners` entry for `internal/reedengine/socketfile_test.go`, since the helper spawns nothing.

## Batch 06 (cards 6–7), head `e887b1d87`

- `lyx loom fmt` formats named Go files and is named in the edit directive.
- A task worktree directory that git deregistered but left refilled is finished with a bounded late-writer retry.

No deviations were reported.
The Windows-only held-open-file row skips on Linux, so it has not run here.

## Verify gate fixes

The gate's `TestEnforcement_FabricVocabulary/tree-scan` found a bare weft/warp token in `internal/gitrepo/pushretry.go`.
The fixer reworded `PushWithRetry`'s doc comment to say "records repository" instead of "the weft" and "the weft's `main`", keeping its meaning, in commit `6d735d6f6`.
It reported `lyx gate test --tags integration ./internal/lyxcwd ./internal/gitrepo` and `lyx loom lint-comments` green.
