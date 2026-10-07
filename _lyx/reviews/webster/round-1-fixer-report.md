# Round 1 fixer report

All three findings fixed, one commit each, nothing pushed.
After the fixes `go build ./...` is clean and `go test` passes over `shuttleengine`, `shedadapters`, `burlerengine`, `loomcli`, `orchengine`, `orchcli`, `stencilstore` and `contracts`.

## Fixed

- **F2 (MEDIUM, design)** — `internal/orchengine/state.go`: `saveStateForStrand` now keeps the persisted `WatcherStopping` when it writes a tick's state, so a tick loaded before the signal no longer erases the stopping record.
  The existing `TestRun_RecordsStoppingBeforeTheInFlightTickFinishes` in `internal/orchengine/watchloop_test.go` now blocks a tick that goes on to read a turn end and save; it failed on the old code and passes on the fix, and no new test was added.
  Commit `42799d076`.
- **F3 (LOW, consistency)** — `contracts/stencils/orch/orch-template-reload.md` and `orch-template-role.md`: the header comments now say a compaction reload types `/reload-plugins` and the pointer with no skills retyped, and that lyx loads the skills after `/clear` only.
  Commit `ff2e7421c`.
- **F1 (BLOCKING, consistency)** — reflowed to one sentence per line, wording unchanged, in `internal/shuttleengine/attach.go`, `internal/shedadapters/burler.go`, `internal/burlerengine/doc.go`, `internal/loomcli/bootstrap.go` and `internal/loomcli/start.go`.
  Commit message `comments: fix F1`, the latest commit on the branch.

## Deferred

Nothing deferred.

## Notes

`gofmt -l` still lists `internal/burlerengine/engine.go` for a misaligned `Round:` field in a struct literal that this diff does not touch; it predates the branch, so I left it.
