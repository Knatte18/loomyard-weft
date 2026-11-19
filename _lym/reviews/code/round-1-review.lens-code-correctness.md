```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the board-viewer diff. I traced the code by hand with concrete inputs and made one tool call.

## Per-function list

- `boardengine.checkExpected`: current revision, stale revision, removed entry and create of an existing slug each give the right result and write nothing. Correct. A project slug has no `Fresh` entry, which is harmless.
- `migrateLoaded` and `Board.View`: a board with an unmigrated entry gives the migrated revision, and nothing is saved. Correct.
- `renderFiles`, `buildView` and `EntryRevision`: the stored copy is used for the revision, and the rendered copy for the decision file. Correct.
- `boardDataScript`: `&lt;/script&gt;` is escaped. Correct.
- `sectionsOccupiedBy`: duplicates are removed, and unknown sections are appended by the page. Correct.
- `FabricReplay.Claims`: `board-data.js` and `board.html` are claimed at the root only. Correct.
- `Server.ServeHTTP`: the Host check, token checks and route table behave. One finding, F2.
- `serveWrite` and `decodeWriteRequest`: a body of `null`, an array, an unknown key, an oversize body and a bad Content-Type or Origin are each refused. Correct.
- `setStatus`, `promote` and `applyWrite`: the revision is checked once outside and once under the write lock. Release, draft and undraft rules match D10. Correct.
- `serveFile`: only names the projection lists are served, and `%2F` paths give 404. Correct.
- `waitForChange` and `beginDrain`: bound, request end and drain all answer unchanged. Correct, except for F1 when the revision is empty.
- `Run`: lock held, bind failure, register failure and state-write failure each release the lock and close the listener. The exit order is: stop taking connections, release the lock, drain, wait for in-flight requests. Correct.
- `watchBoard`: the timer fallback works, and a closed event channel is set to nil. One finding, F3.
- `contentSecurityPolicy`: hashes the inline script and style, skips the `src` script tag, and the page JS holds no `&lt;/script&gt;`. Correct.
- `runOpen`, `awaitServer` and `answeringState`: a held lock with a stale state, a free lock with a leftover state, a failing spawn, a failing prime, an opener error and the cap all give the right result. Correct.
- `spawnServer` and `waitOnOpener`: the child is reaped and the opener exit reaches the caller. Correct.
- `refuseBoardVerb`: the run-agent check comes before the `--board-path` check. Correct.
- `BuildWorkspace` members and `regenerateHubWorkspace`: refusals are made before any write, members are sorted, and the beam-anchor failure only warns. Correct.
- `WorkspaceContent` and `WorkspaceTasks`: an empty beam becomes `root`, and an empty lym path becomes `lym`. Correct.
- `writeWorktreeTasks` and the keybindings block: correct.
- `board.html` `safeTarget` and `markdownToHtml`: `javascript:`, `//host`, `/\host`, tab-split schemes and entity tricks all render as plain text or stay relative. Correct.

## Findings

**F1: MEDIUM, design.** The page busy-loops when `board.json` is absent.
- Location: `internal/boardengine/board.html:966-982` (`poll`) with `internal/boardserve/server.go:245` (`waitForChange`).
- With no `board.json`, `BoardRevision` returns `""` and the server answers `revision:""`.
- `poll("")` sends no `after` parameter, and the server treats an empty `after` as "answer at once".
- The page then re-requests with no delay and re-renders every time, because `unchanged` is false.
- Each request also resets the idle timer.
- Fix: give the absent board a non-empty sentinel revision, or send `after=` whenever the revision is a string and let the server tell "absent" from "no after".

**F2: LOW, design.** An unauthenticated `/api` request resets the idle clock.
- Location: `internal/boardserve/server.go:140-144`.
- `lastAPI` is stamped before the token check, so any local process or wrong-token request keeps the server alive past D11's 30 minutes.
- Fix: stamp `lastAPI` only after `bearerAuthorized` succeeds on `/api/board` and `/api/write`.

**F3: MEDIUM, design.** The watch loop has no one-second floor on event wakes.
- Location: `internal/boardserve/run.go:424-433`.
- Evidence: `internal/fswatch/doc.go:10` says the package coalesces and debounces nothing and "the caller owns timing".
- The loop wakes on every event and then reads and hashes `board.json`. A burst of events produces a burst of wakes, which D11 and `PATTERN-daemon-wakeups` rule out.
- Fix: after an event wake, wait at least one second (via `opts.After`, so a test can inject it) before the next read, or drain events that arrived in that window.

**F4: LOW, consistency.** The state file may be briefly group-readable.
- Location: `internal/boardserve/state.go:79-85`.
- `state.WriteFile` and `WriteJSON` do not set a mode. I grepped `internal/state/state.go` for explicit modes and found only `0o755` for directories, so the file gets the default creation mode.
- `hardenFile` chmods only afterwards, so the token is exposed between the write and the chmod. D9 asks for 0600 as written. The test only checks the mode after the write.
- Fix: create the state file at 0600 (write a 0600 temp file in the state directory before renaming), or make the state directory 0700.

**F5: LOW, design.** A create request that carries a `revision` is treated as an edit.
- Location: `internal/boardserve/write.go:289`.
- `checkRequest` requires a revision for non-create ops but does not forbid one on create, so `Expecting(slug, nonEmpty)` turns a create of an existing slug into an edit.
- Fix: refuse a non-empty revision on `create`.

## Tool calls

1. `grep` over `internal/state/*.go` and `internal/fswatch/*.go`: the file modes `state.WriteJSON` uses and whether `fswatch` debounces. Neither package was in my context.
