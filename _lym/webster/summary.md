# Board viewer, its server and the VS Code keys

All three batches recorded `done`; no recovery, plan fix or gate fixer ran.
Each fork ran the tier0 and tier1 gates only.

## Batch 01-board-view-projection (cards 1–4)

Head `adbe98d3`.
Built the `board-data.js` view projection with entry and board revisions and `Board.View`, the stale entry-revision refusal inside the write section, the embedded `board.html` claimed in the Bolt replay, and the `boardserve` HTTP handler.
Deviations reported by the scope guard: `cmd/lym/testonlycode_linux_test.go`, `internal/boardengine/expect.go`, `expect_test.go`, `view.go`, `view_test.go`, `internal/boardserve/server.go`, `write.go`.
Departures: no `Pattern-Departure:` trailer in the digest; the fork reports its write decoder refuses unknown keys through a key map rather than `DisallowUnknownFields`, because `PATTERN-tolerant-records` bars that call.
Renames: none.
Review-only: `board.html` was not syntax-checked, since `node` was unavailable.

## Batch 05-board-server-lifecycle (cards 5–6)

Head `35fbb9e2`.
Built the single-instance lock, state, launcher, watch loop and idle and binary exits, plus `lym board serve` and `lym board open`, the opener and the `runproc` role.
Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/boardcli/serveopen.go`, `internal/boardserve/run.go`, `state.go`.
Departures: none.
Renames: none.

## Batch 07-vscode-tasks-and-keys (cards 7–9)

Head `ffe4c65c`.
Built the `board open` worktree task and the alt+shift+9 and alt+shift+0 keys, fabric's opaque workspace-content hook with vscode's member splice, and the ide hook writing the `beam start` task into the hub and beam workspace files.
Deviations: `internal/fabricengine/hubworkspace.go`, `hubworkspace_integration_test.go`, `internal/ideengine/spawn_hub_integration_test.go`, `internal/vscode/config.go`, `config_test.go`.
Departures: none.
Renames: none.
