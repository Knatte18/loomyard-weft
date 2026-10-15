# Show the orch context size in the batten overview

The prime's overview window (`lyx batten overview`) now opens with one line for the orch session, above the run lines.
The operator sees the orch's context size beside the batten runs without running `lyx orch status`.

The line reads `orch  <tokens> / soft <soft> / hard <hard>  <phase>  cycles <n>`:

- Token counts render in rounded-down thousands with a `k` suffix (`212k`); a count under 1000 renders as the plain number.
- An unknown reading, as after a fresh launch before the first turn end, renders the tokens as `-`.
- The phase is the persisted `Phase` value as it is, so a reading during a non-idle phase may predate the cycle.
- A non-empty `WatcherExit` appends `watcher down`, so a cleanly stopped watcher's stale reading does not look current.
  A watcher killed outright records no exit and shows no marker.
- An absent state file draws no orch line, and the frame equals the one before this change.
- With no run in flight the frame is the orch line, then `no runs in flight`.
- The run lines' columns and ordering are unchanged.

## Design decisions

- `orchengine` gains `PeekState`, which decodes the state file at `Paths.StatePath` with no lock and no mkdir; an absent file reads as not found.
  `LoadState` and `state.ReadJSONNoMkdir` take a read lock that creates the lock file, so neither fits a pane that must write nothing and never block the watcher.
  `internal/state` writes atomically, so a lockless read never sees a torn file.
  The decode stays in `orchengine` so the orch state keeps one owner of its shape.
- The overview reads the state once per frame with the same tolerance as the run lines' `statuspane.StatusCache`.
  One failed read, including a decode failure, keeps the last good line; a second failure in a row draws `orch  state unreadable`.
  The run lines draw in either case.
- Thresholds come from `orchengine.LoadConfig` with the orch config, loaded once when the overview starts.
  A load failure is logged at Warn and the line shows `-` for both thresholds; it does not refuse the verb.
  A threshold edit therefore shows only once the runs strand is replaced.
- `battencli` already imports `orchcli` and `orchengine`, so the read adds no import edge.
- The orch line code is in the new `internal/battencli/overvieworch.go`.
  The verb's help text and the `orchengine` package doc now mention the orch line.
  `cmd/lyx/testdata/test-budget.yaml` is raised for the new tests.

## To check by hand

- `run-pane-timing` also edits `internal/battencli/overview.go` to number the run lines' stage cells; whichever lands second rebases on the other.
- The `testtiming:keep` directive above `TestCycleRequest_WriteProbeClear` was detached by a review fix and moved back; check it sits directly above that test and no other test lost its directive.
- The overview was not run against a live orch in a prime; check the line in a real overview window, including after `lyx orch stop`.
