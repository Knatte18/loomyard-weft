---
verdict: APPROVED
---

## Alignment at pr-gate: reed-down-and-pane-path

The diff of pull request 693 at `c177c9eb8` against `main` matches the board entry's goal for #679 and #680 and the decision record's D1 through D8.

- D1: `Engine.Down` writes `DiscoverStopMarker` atomically into the told discover signal path only on its server-teardown branch, before kill-server, and reports `DownResult.EndedServer`; standalone mode and a live sibling write nothing (`internal/reedengine/lifecycle.go`).
- D2: `consumeDiscoverSignal` reads before it removes, a stop-woken cycle uses the 1-second stop floor, and an idle stop-woken listing returns at once (`internal/reedcli/watchdog.go`).
- D3: `down` polls the watchdog lock for at most 10 seconds when `EndedServer` is set, logs one Warn on timeout, keeps its envelope, and has injectable wait, poll, probe and sleep (`internal/reedcli/up.go`).
- D3a: after a clean loop exit the daemon releases its lock, skips the re-check on a cancelled context, and spawns a successor only on an affirmative listing.
- D4: the replaced-binary check runs after the cycle's listing, and an idle listing ends the daemon with no successor of its own.
- D5: `bash-env.sh` keeps `exec </dev/null` and prepends `LYX_BIN`'s quoted parent directory, with a `cygpath -u` variant on Windows that does nothing when `cygpath` is absent.
- D6: `agentname.LyxBinEnv` replaces reedengine's unexported key, and both the prelude and the Claude engine read it.
- D7: the prelude is unchanged and `internal/reedengine/doc.go` documents the fresh-shell case.
- D8: the three suites' pre-condition bullets name the shell each check runs in, and the creel suite's closing sentence counts only the remaining checks.

The docs the Scope names, `internal/reedcli/doc.go`, `internal/reedengine/doc.go`, the `down` and `watchdog` help and the Claude engine's `doc.go`, change in the same diff.
No change falls outside the Scope, and none of the Scope's Out items is touched.

The open risks and the carried-over review findings were read as part of the run.
They describe residual behaviour the record accepts: a reap-time `down` can hit the 10-second warn, D5 depends on Claude Code's env file ordering, and a free lock does not mean the exiting process is gone.
None breaks a recorded decision.
The record's risk about `creel-stage1-fixes` editing `SANDBOX-CREEL-SUITE.md` has no counterpart in flight: that slug is not among the peers, and the branch's merge of `main` at `324d562` already carries the parent's lines.

## Overlaps

- `webster-fork-stencil-fixes` (row 13, Webster-Bouncer): the only shared path is `cmd/lyx/testdata/test-budget.yaml`, a shared bookkeeping file, where this run changes only the `internal/reedcli` line.
  It is not a design overlap.
  Landing order: this run (row 22) is further along by the recipe and lands first; the peer takes the change at Merge-In.
- Parent commits: `lyx loom peers` lists none.

## Superseded

No board entry is made obsolete by this run beyond its own.
