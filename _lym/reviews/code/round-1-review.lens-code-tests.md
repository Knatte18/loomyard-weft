```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none. I made no tool calls.

**Per behaviour: pinning test**

Board engine:
- Stale check under the write lock, before the mutation, with nothing written: `TestExpectingRevision` in `expect_test.go`. Rows cover a current revision, a status-only stale revision, a migrated board, a removed entry, an existing slug on create, and a new slug.
- The "nothing saved, rendered or synced" outcome is pinned through `assertSameFiles(dataFiles)`. The board has git skipped, so "no sync" is not directly observed.
- CLI verbs stay revision-free: the existing CLI write tests, unchanged.
- Projection placement (section, project handle, waits_on) matches the README: `TestRenderBoardData/EntriesStandWhereTheReadmeShowsThem`.
- Projects with parents, sections, type/area split and the revision per entry: `TestRenderBoardData`, rows `ProjectsList…`, `HeaderAndSections` and `EntriesCarryTheirStoredValues`.
- `&lt;/script&gt;` and `&lt;b&gt;` escaped: `ABodyCannotEndTheScript`.
- Revision taken over the stored entry, not the Discussed-clearing copy: `RenderToDiskClearsAMissingDecisionRecord…`.
- `BoardRevision`: its own row, with an independent sha256.
- `View` over a migrated board, writing nothing: `ViewOfAMigratedBoard…`.
- `RenderToDisk` writes both files and the manifest lists them: `TestRenderToDiskManifestCleanup`.
- The replay claims both names, and `sub/board.html` is not claimed: `TestFabricReplay_Ledger`.
- `Config.Outputs.Labels`: `TestOutputs`.
- `board.html` equals the embedded bytes, and the page loads `board-data.js` with no `http(s)://` URL: `TestViewerPage`.
- Owner-set write path commits both new names: none.
- Both files land in the sync commit beside the README (D3 acceptance): none.
- The JS page: exempt, there is no JS runtime.

Board server:
- Wrong Host, no or wrong token, no JSON content type, foreign Origin, body over the cap, unknown request key, unknown route, and the file route with no or a wrong `t`: `TestServer/TransportRefusalsChangeNothing`. It also checks the no-store, nosniff and no-CORS headers.
- CSP hashes and no `unsafe-inline`; `GET /` returns the page bytes: `PageCarriesAPolicy…`.
- `GET /api/board` equals `Board.View` and stamps API activity: `BoardReadIsTheProjection`.
- The six writes match the CLI twin board and answer the fresh entry: `PageWritesLeaveTheBoardAsTheCLICallsDo`.
- Edit with the revision a write returned: `EditWithTheRevisionAWriteAnswered…`.
- Draft, undraft, release and promote refusals (including release of a run-status task naming `set-status`), no revision, unknown op and unknown field: `RefusedWritesChangeNothing`.
- Run-lock message on edit: `EditOfARunTask…`.
- Stale edit/release, create of an existing slug (409 with the fresh entry), and removed entry (409 with explicit null): `StaleWritesAnswerTheFreshEntry`.
- Long poll on bound, on change, and on drain: three `LongPoll…` and `DrainAnswers…` rows.
- `/file/&lt;name&gt;` serves only listed names, with a 404 for `a/b.md` and `board.json`: `FileRouteServes…`.
- Single instance, leftover state and log, pid log line, 32-hex token, Register called once, state and launcher at 0600, launcher with 127.0.0.1 and the token after `#`, exit on replacement: `TestRun/ServesOverLeftovers…`.
- `ErrRunning` writes nothing: `HeldLockReturnsErrRunning…`.
- Missing state directory is not created: `MissingStateDirectory…`.
- Idle exit measured from the last API request: `IdleBoundIsMeasured…`.
- Write in flight completes, and the lock is released before the drain: `WriteInFlightCompletes…`.
- Exit answers a waiting poll "unchanged": `ExitAnswersAWaitingLongPollUnchanged`.
- Watch event wakes a poll, and the timer fallback works when the watch fails: `watchTests` (two rows).
- Windows owner-only DACL: `TestHardenFileGrantsOnlyTheOwner`.
- Watch timer doubling from 1 s to 8 s and reset on change: none.
- Production `Register` closure under `HubOwner` with `RoleBoardServer`: none. The test uses a fake `Register`.

Board CLI:
- `runOpen` paths: live server reused, mid-wait state, held lock with no answer (refused naming the log), free lock spawning in the prime, never answering, prime error, spawn error, failing opener, opener cap, no cap: `TestRunOpen`.
- `--board-path` and run-agent refusals for both verbs, with their ways forward: `TestServeOpenRefusals`.
- Real spawn with the prime cwd, a second open starting nothing, serve beside a running server, no token in the address, the launcher handed to the opener: `TestBoardServeScenario` (Linux, tier 1).
- Orch strand or no strand name proceeds past the refusal (D12): none.
- `productionOpenDeps` sets `OpenerCap` from `KeepWarm` for a strand-named caller: none.
- Server board is built with an empty strand name (D12): none. The scenario sets an empty strand env, so it cannot see this.

Fabric, IDE and VS Code:
- No hook means no `tasks` key: `TestRegenerateHubWorkspace/CarriesNoTasksKeyWithNoHookInstalled`.
- The hook receives slug and anchor, and the members are spliced: `HandsTheHook…`.
- Refusal of `lym.generated`, `folders` and `settings` writes no file: `RefusesAHookMember…`.
- `BuildWorkspace` member splice, ordering and refusals: `TestBuildWorkspace/Members_*`.
- `WorkspaceTasks` shape: `TestWorkspaceTasks`.
- `WorkspaceContent`: root for an empty beam, the beam slug, and the bare `lym` fallback: `TestWorkspaceContent`.
- Hub file runs `beam start root` and beam file runs `beam start &lt;beam&gt;`: `assertBeamStartTask`.
- The worktree `tasks.json` holds exactly `reed attach` and `board open`, so no `beam start`: `assertWorktreeTasks`.
- `board open` presentation `reveal: always`, `focus: false`: `assertBoardOpenTask`.
- Bare-lym fallback in `WriteConfig`: `TestWriteConfigFallsBackToBareName`.
- Keybinding block entries and a second seed leaving one of each: `TestKeybindingsBlock` and `TestSeedKeybindings`.
- `InstallWorkspaceContent` called in main: `TestPersistentPreRunE_InstallsSeams`.
- `RoleBoardServer` ended with the hub's processes, before the dobby: `TestEndSubject_…`.
- Beam anchor unreadable, so the file gets no hook members (warn branch in `hubworkspace.go`): none.

**Findings**

1. **MEDIUM, design.** `internal/boardcli/serveopen.go` (`refuseBoardVerb`, `serveCommand`). D12 acceptance says an orch strand name or none proceeds, and the server passes an empty strand name. No test runs either verb with an orch strand name past the refusal. No test checks that `serveCommand` clears `boardConfig.StrandName`. Evidence: `serveopen_test.go` only runs `--board-path` with an empty or run-agent strand.
2. **MEDIUM, design.** `internal/boardcli/serveopen.go` (`productionOpenDeps`). The `KeepWarm` cap wiring for a strand-named caller has no test. `TestRunOpen` injects `OpenerCap` straight into `openDeps`. A broken strand-to-cap mapping would pass.
3. **MEDIUM, design.** D3 acceptance: a board write's sync commit holds `board-data.js` and `board.html` beside the README, and the owner-set write path commits every name `Render` returns. No test in the diff covers the commit contents or the owner-set path. The diff touches only render and replay tests. `render_test.go` and `replay_test.go` show only the manifest and the claims.
4. **LOW, design.** `internal/boardserve/run.go:watchBoard`. The 1 s to 8 s doubling and the reset to 1 s on change are not asserted. The fake clock never records the `After` durations. Suggested assertion: the durations `After` was called with.
5. **LOW, design.** `internal/fabricengine/hubworkspace.go`. The warn branch for an unreadable beam anchor (no hook members) has no row.
6. **LOW, design.** `internal/boardserve/run.go`, `internal/boardcli/serveopen.go` (`Register`). The production `Register` closure (`HubOwner`, `RoleBoardServer`) has no test. Only the fake is covered.
7. **LOW, design.** `internal/boardengine/view_test.go`, `EntriesCarryTheirStoredValues`. It computes the expected revision with `EntryRevision` itself, so the test is tautological. The D8 definition (hex SHA-256 of the entry's JSON) is pinned nowhere. Assert `sha256(json.Marshal(task))` independently.
8. **LOW, consistency.** `internal/boardserve/server_test.go`, `PageHoldsEachRoute`. It is a new assertion on the static prose of `board.html` that no acceptance criterion asks for.
9. **LOW, consistency.** `TestViewerPage` is a redundancy candidate, but its byte-equality assertion for `board.html` on an empty and a changed board is not asserted by `TestRenderBoardData` or the server's `GET /` test. Mark it `//testtiming:keep pins that every render writes the embedded page bytes`. I could not see `TestFileRaddle`, so I could not compare it.
10. **LOW, consistency.** `TestKeybindingsBlock` asserts the exact block text, which `TestMergeKeybindings` is unlikely to assert. Mark it `//testtiming:keep pins the block's entry text`. I could not see `TestMergeKeybindings`.

No assertion the diff loosened was visible in the changed tests. The task-count checks went from 1 to 2, and the `Fatalf` and `assertBoardOpenTask` additions are stricter.

**Tool calls:** none made, since the inherited context was enough.
