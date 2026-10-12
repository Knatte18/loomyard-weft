## Changes

- F1: fixed as proposed. The `lyx webster run` help in `internal/webstercli/run.go` no longer lists clearing a leftover pause flag.
- F2: fixed as proposed, comments only. Dropped `pause.go` from `internal/websterengine/outcome.go`, the pause-clear clause from `internal/websterengine/runlevel.go`, and `pause` from the verb lists in `internal/webstercli/cli.go`, `wiring.go` and `cli_test.go` (both sites). `internal/webstercli/sync.go` now names treadle's round-loop pause flag as the one remaining.
- Gate repair: the F2 edits left fixed-column-wrapped comment lines in `cli.go`, `sync.go`, `wiring.go` and `runlevel.go`. Those comment paragraphs are re-broken at sentence boundaries, one sentence per line, with no wording change beyond dropping the `--` aside in `sync.go`'s second sentence into its own line.

- Second gate repair: the `runlevel.go` file header is one sentence, so it is now a single line instead of clause fragments split at commas.

`lyx gate test ./internal/webstercli ./internal/websterengine` passes. F1 and F2 are one commit each; the two gate repairs are the others.

## Disputed

None.

## Deferred

None.
