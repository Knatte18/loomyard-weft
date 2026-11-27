```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, mapping each behaviour the diff adds or changes to the test row that would fail if it broke.

**Behaviour → pinning test**

Projection, revision, render:
- `board-data.js` statement, placement, projects and parents, sections, types/areas/owner/render time: `TestRenderBoardData`
- HTML escaping, type/area split, recipe, owner, dates, `design_file`: `CarriesEachEntrysFields`
- `status` and `kind`: indirectly, through the `TestServer` refusal rows
- `decision_file` named when the record exists: **none** (F4)
- `waits_on` slug fallback for a dependency with no handle: **none** (F9)
- Revision over the stored entry; `BoardRevision`; `View` migrates and writes nothing: `TestRenderBoardData`
- `Outputs.Labels`: `TestOutputs`
- `board.html` bytes, script tag, no remote URL: `TestViewerPage`; `GET /` in `PageCarriesAHashOnlyPolicyAndTheRoutes`
- Both files written and manifested: `TestRenderToDiskManifestCleanup`; replay claims: `TestFabricReplay_Ledger`
- Stale, removed, create-existing, create-new and unmigrated revision: `TestExpectingRevision`

Server handler:
- Host, token, Content-Type, foreign Origin, body cap, unknown key: `TransportRefusalsChangeNothing`
- The server's own Origin is accepted: **none** (F1)
- Response headers, CSP, routes in the page: `fixture.serve`, `PageCarries…`
- View answer: `BoardAnswersTheView`
- Long poll bound, change, drain: `LongPoll`
- Long poll with an empty `after` (no `board.json`): **none** (F2)
- File route serves a design file and 404s others: `FileRouteServesOnlyListedFiles`
- File route serves a decision file or a project design file: **none** (F4)
- Six ops, op refusals, run lock: `EachOpWritesWhatItsVerbWrites`, `BoardRefusalsChangeNothing`
- Stale create, edit, release: `StaleWritesAnswerAConflict`
- Stale promote: **none** (F3)

Lifecycle:
- Held lock, leftovers, state and launcher, idle exit, replaced binary, in-flight write, event wake and floor, timer backoff: `TestRun`
- Windows DACL: `TestHardenFileGrantsOnlyTheOwner`

Verbs:
- `runOpen` paths, cap and no cap: `TestRunOpen`
- Run-agent and `--board-path` refusals: `TestServeOpenRefusals`
- Prime cwd, launcher handed to the opener, second open, second serve: `TestBoardServeScenario`
- Keep-warm cap chosen by strand name in `openCmd`: **none** (F5)
- `serve` registers `RoleBoardServer` under `HubOwner`: **none** (F6)
- `serve` empties the strand name: **none** (F7)
- Role not spared by `EndSubject`: `TestEndSubject_…`

VS Code and fabric:
- Worktree tasks, keybindings, `WorkspaceTasks`, `WorkspaceContent`, member splice and refusals: their tests and the ide checks
- Hook install: `TestPersistentPreRunE_InstallsSeams`; hook arguments, no hook, refusal writes nothing: `TestRegenerateHubWorkspace`
- Beam with an unreadable anchor still gets its file, with no hook member: **none** (F8)
- Daemon-wakeups listing: `daemonwakeups_test`

The page's in-browser behaviour is a review check, since there is no JavaScript runtime.

**Findings**

- **F1.** MEDIUM, `design`. `internal/boardserve/server.go` (`serveWrite`, Origin check, about line 247).
  - No test sends the server's own `Origin`. A browser sends it on every page POST.
  - If the comparison refused it (wrong scheme or host string), every page write would fail and all tests would pass.
  - Evidence: `internal/boardserve/server_test.go`, `TransportRefusalsChangeNothing` has only `ForeignOrigin`, and `request()` sets no `Origin`.
- **F2.** MEDIUM, `design`. `internal/boardserve/server.go` (`serveBoard`, `query.Has("after")`, about line 174).
  - No test long-polls with `after=` on a board with no `board.json`.
  - Reverting to a non-empty-value check would bring back the page's busy loop with no test failing.
  - Evidence: `server_test.go` `LongPoll` and every `run_test.go` poll use a seeded board with a non-empty revision.
- **F3.** MEDIUM, `design`. `internal/boardserve/write.go` (`promote`, about line 150).
  - No test sends a promote with a stale revision.
  - Promote carries its own `Expecting(slug, revision)`. Dropping it would let a stale page promote (and, for a discussed Sketch, release it) with every test passing.
  - Evidence: `server_test.go` `StaleWritesAnswerAConflict` covers create, edit and release only; the promote row in the ops table uses the current revision.
- **F4.** MEDIUM, `design`. `internal/boardengine/view.go` (`buildView`, `entry.DecisionFile`) and `internal/boardserve/server.go` (`listsFile`, about line 236).
  - No test asserts that an entry with a stored record names its `decision_file`, or that `/file/` serves it or a project's design file.
  - Never setting `DecisionFile`, or dropping either `listsFile` branch, fails nothing.
  - Evidence: `view_test.go` asserts only `DecisionFile == ""` for the missing record; `FileRouteServesOnlyListedFiles` fetches only `design-open.md`.
- **F5.** MEDIUM, `design`. `internal/boardcli/cli.go` (`openCmd`, `openerCap` selection).
  - Nothing pins that a strand-named caller gets the keep-warm cap and the operator gets none.
  - Dropping the assignment would block an orch's `open` until the browser exits.
  - Evidence: `open_test.go` injects `OpenerCap` directly. `serveopen_test.go` stops at the `--board-path` refusal. The Linux scenario sets an empty strand name.
- **F6.** MEDIUM, `design`. `internal/boardcli/cli.go` (`serveCmd`, `Register`).
  - No test shows that a real `serve` writes a `RoleBoardServer` record under `runproc.HubOwner`. Without it a takeover would not end the server.
  - Evidence: `run_test.go` injects a counting fake. `runproc/end_test.go` plants its own record. `serve_linux_integration_test.go` never reads the run-process directory.
- **F7.** LOW, `design`. `internal/boardcli/cli.go` (`serveCmd`, `served.StrandName = ""`).
  - No test shows that page writes carry an empty strand name when an orch session started the server (D12).
  - Evidence: no test writes through a spawned server, and the scenario sets `LYM_STRAND_NAME=` empty anyway.
- **F8.** LOW, `design`. `internal/fabricengine/hubworkspace.go` (`regenerateHubWorkspace`, `NamedWorktree` error branch).
  - No test covers a beam whose anchor cannot be read: its file should still be written, without hook members.
  - Evidence: the new `hubworkspace_integration_test.go` rows cover no hook, hook arguments and refused members only.
- **F9.** LOW, `design`. `internal/boardengine/view.go` (`buildView`, `waits_on` loop).
  - The slug fallback for a dependency the board holds no handle for is unpinned in the projection.
  - Evidence: `view_test.go` `viewFixture` has only dependencies on `base` and `shipped`, both with handles.

Tool calls: none.
