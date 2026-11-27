```yaml
lens: code-tests
outcome: reported
```

Task: lens `code-tests`, mapping each behaviour the diff adds or changes to the test row that would fail if it broke.

**Behaviour → pinning test**

| Behaviour | Test |
|---|---|
| `board-data.js` wrapper, placement, sections, parents, types/areas, owner, dates, recipe, design file | `TestRenderBoardData/MatchesTheReadme` |
| Script-closing text escaped | same, but only asserts absence (F1) |
| `decision_file` named when the record exists | **none** (F2) |
| Stored-entry revision with a missing record; `BoardRevision`; `Board.View` migrates and saves nothing | other `TestRenderBoardData` rows |
| `Outputs.Labels` | `TestOutputs` |
| Manifest lists both new files; replay claims them | `TestRenderToDiskManifestCleanup`, `TestFabricReplay_Ledger` |
| Page rendered verbatim, no remote URL; page browser behaviour | `TestViewerPage`; browser behaviour is review-only |
| Revision check (saved, changed, removed, existing, new, migrated) | `TestExpectingRevision`, `TestServer` |
| Host, token, content type, origin, body cap, unknown key, headers, CSP | `TestServer` |
| Wrong method refused with 405 | **none** (F10) |
| Six ops, op refusals, 409s, run lock | `TestServer` |
| Release, draft or undraft of a project; writes whose target is a project | **none** (F5) |
| Empty slug refused | **none** (F10) |
| Long poll: bound, change, drain | `TestServer/LongPoll`, `TestRun` |
| File route serves a listed entry design file, 404s others | `TestServer/Files` |
| File route serves a decision file or a project design file | **none** (F3) |
| Single instance, leftover state and log, token, modes, launcher, `Live` | `TestRun` |
| Idle exit, replaced exit, write completes on exit, lock released | `TestRun` |
| No new connection once exit begins | **none** (F9) |
| Watch event or timer wakes a poll | `TestRun` |
| Timer goes from 1s, doubling to 8s, and resets on change | **none** (F4) |
| Free lock acquired with zero wait | `TestAcquireWriteLockWithin` |
| `runOpen` paths | `TestRunOpen` |
| `--board-path` and run-agent refusals | `TestServeOpenRefusals` |
| Orch strand name passes both verbs | **none** (F8) |
| Strand name selects the keep-warm cap | **none** (F6) |
| `serve` registers under `runproc.HubOwner` | **none** (F7) |
| `serve` empties the strand name | none; no observable difference for the names that pass the refusal, so no finding |
| Real spawn, prime cwd, second open, rival serve, opener argument | `TestBoardServeScenario` |
| `RoleBoardServer` ended at takeover | `TestEndSubject_...` |
| Worktree tasks and keys | `vscode` config and keybinding tests, ide checks |
| Workspace members spliced, refusals, nothing written, hook arguments, no hook | `TestBuildWorkspace`, `TestRegenerateHubWorkspace` |
| Unreadable beam anchor writes the file with no hook member | **none** (F11) |
| `beam start` task and hook install | `TestWorkspaceTasks`, `TestWorkspaceContent`, ide checks, `TestPersistentPreRunE_InstallsSeams` |

**Findings**

- **F1. MEDIUM, design.** `internal/boardengine/view_test.go:153`
  - The escaping check only asserts that no literal `</script>` or `<b>` is in the file. It still passes if the projection drops `Body` altogether.
  - No row asserts any `ViewEntry.Body`, `Title`, `Brief` or `Handle` value, or a project's title, brief or labels. `TestServer/Board` compares the answer to `View()` itself.
  - Fix: assert the decoded body equals `</script><b>bold</b>`, plus literal values for the other displayed fields.
- **F2. MEDIUM, design.** `internal/boardengine/view_test.go:182`
  - Only the empty `decision_file` case is asserted. If `viewEntry` never set `DecisionFile`, every test passes and the page loses its decision-record link (D1, D4).
  - Fix: assert `held.DecisionFile == "decision-held.md"` on the `Render` path.
- **F3. MEDIUM, design.** `internal/boardserve/server.go:242-244`
  - The file route's `DecisionFile` branch and project `DesignFile` branch have no row. `TestServer/Files` (`server_test.go:442`) serves only `design-drafted.md`.
  - Removing either branch turns those links into 404s unnoticed.
- **F4. MEDIUM, design.** `internal/boardserve/run.go:183-189`
  - The watch timer's backoff (D11, `PATTERN-daemon-wakeups`) is unpinned. `stepClock.After` (`run_test.go:53`) ignores the requested duration.
  - A fixed or never-reset interval passes every test, and the scan only sees sub-second literals.
  - Fix: record the requested durations and assert 1s, 2s, 4s, 8s, 8s, then 1s after a change.
- **F5. LOW, design.** `internal/boardserve/write.go:163`
  - Release, draft and undraft of a project have no row; only notes are tested.
  - No row covers a write whose target is a project through `checkExpected` either: a stale edit with a null `Fresh`, or a success with a null entry.
- **F6. MEDIUM, design.** `internal/boardcli/cli.go:463-465`
  - Nothing fails if the strand-name → `KeepWarm(...).Cap` wiring is removed or inverted. An agent's `open` would then wait on the browser without bound (D5).
  - `TestRunOpen` injects `OpenerCap` directly, and the scenario sets an empty strand name.
- **F7. MEDIUM, design.** `internal/boardcli/cli.go:498-500`
  - No test checks that `serve` registers a `runproc` record under `HubOwner` with `RoleBoardServer`.
  - `TestRun` uses a counting fake, and `TestEndSubject_...` plants its own record. With the registration gone, a hub takeover would leave the server running (D11).
  - Fix: read the record in `TestBoardServeScenario`.
- **F8. LOW, design.** `internal/boardcli/serveopen_test.go:24`
  - D12's criterion says an orch's strand name proceeds. The rows cover only an empty name and a run agent.
  - Fix: add an orch row that reaches the `--board-path` refusal.
- **F9. LOW, design.** `internal/boardserve/run.go:139-143`
  - Nothing asserts that a new request is refused once exit begins. `open` relies on this to tell a draining server from a live one (D5, D11).
  - Fix: in `ExitAnswersPollsAndCompletesTheWriteInFlight`, assert `Probe(server)` fails while the write is still blocked.
- **F10. LOW, design.** `internal/boardserve/server.go:298`, `internal/boardserve/write.go:79`
  - The 405 refusal for a wrong method and the empty-slug refusal have no row. Both are input checks at a trust boundary; add them to the existing refusal tables.
- **F11. LOW, design.** `internal/fabricengine/hubworkspace.go:236`
  - The case where a beam's anchor cannot be read (Warn, file still written without hook members) has no row.

Tool calls (6 of 6):
1. `sed` of `server_test.go` lines 140–480: the context held only its head.
2. `cat run_test.go`: the context held only its outline.
3. `cat open_test.go` and `expected_test.go`: only outlines were held.
4. `cat view_test.go` and `viewer_test.go`, plus the diffs of the render, replay and config tests: not held.
5. Diffs of the `fabricengine`, `ideengine` and `vscode` tests: not held.
6. `grep` for line numbers to locate the findings: the diffs in context carried none.
