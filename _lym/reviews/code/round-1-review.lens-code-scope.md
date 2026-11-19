```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each change in the diff to a decision, and each decision to the changes that build it.

## Per change: the decision it traces to

| Change | Decision |
|---|---|
| `boardengine/view.go`, `config.go` (`Outputs.Labels`), `render.go` (`renderFiles`, `unmetDependencies`), `Board.View`, `migrateLoaded` | D1, D8 |
| `boardengine/viewer.go`, `board.html` | D2, D4 |
| `render.go` (`BoardDataFile` and `BoardPageFile` in the file map, so the manifest lists them), `replay.go` `Claims` | D3 |
| `boardengine/expect.go`, `board.go` (`checkExpected`, `Expecting`) | D8 |
| `taskguard.go` (`IsRunAgentSession` exported) | D12, forced by D5 and D11 |
| `boardserve/server.go`, `write.go` | D9, D10 |
| `boardserve/run.go`, `state.go`, `owneronly_*.go`, `lymdirs.BoardServeDirName`, `runproc.RoleBoardServer` | D11, D9 (owner-only files) |
| `boardcli/serveopen.go`, `opener_*.go`, `cli.go` wiring | D5, D11, D12 |
| `vscode/config.go` (`board open` task), `keybindings.go` | D6 |
| `vscode.WorkspaceTasks`, `BuildWorkspace` members, `fabricengine` `InstallWorkspaceContent` and `workspaceMembers`, `ideengine.WorkspaceContent`, `cmd/lym/main.go` install | D6, and the arbiter's D6 seam entry |
| `cmd/lym/daemonwakeups_test.go`, `pattern/PATTERN-daemon-wakeups.md` | D11 and its `PATTERN-daemon-wakeups` constraint, a declared change |
| `docs/overview.md`, `boardcli`, `boardserve`, `boardengine`, `fabricengine`, `runproc` doc.go, `rerender` Short text | Forced by the traced changes: stale docs, and the repository's rule that docs land in the same diff |
| Test edits in `ideengine`, `vscode`, `runproc` and `cmd/lym/prerunseams_test.go` | Forced by D6, D11 and the new seam |

No change traces to nothing. I found no unrequested scope, and no `Pattern-Departure:` trailer is needed: the commit messages carry none.

## Per decision: the changes that build it

- **D1:** `view.go`, `render.go`, `config.go`.
- **D2:** `viewer.go`, `board.html`.
- **D3:** `render.go`, `replay.go`. The owner-set write path commits every name `Render` returns, so it needed no edit of its own.
- **D4:** `board.html`.
- **D5:** `serveopen.go`, `opener_linux.go`, `opener_windows.go`, `cli.go`.
- **D6:** `vscode/*`, `fabricengine/hubworkspace.go`, `ideengine/spawn.go`, `main.go`.
- **D8:** `expect.go`, `board.go`.
- **D9:** `server.go`, `state.go`, `owneronly_*.go`.
- **D10:** `server.go`, `write.go`.
- **D11:** `run.go`, `state.go`, `lymdirs`, `runproc`, the `PATTERN-daemon-wakeups` entry.
- **D12:** `serveopen.go`, `taskguard.go`.
- **D13:** the new `boardserve` package, `boardcli` and `runproc`.
- **Added after Discussion entries:**
  - Per-body README folds moot: nothing was built, which is correct.
  - No `launch.json`: none written.
  - Phone goal met, and the prime's open chain and LY title stay out: none built.
  - D6 seam placement: built as the entry says.
  - Render outputs stay committed: built.
  - Open's `rerender` refusal dropped: `runOpen` never reads `board.html` or `board-data.js`.
  - Opener waited on through `cmd.Wait`: `waitOnOpener`.

No decision lacks a building change.

## Findings

None.

One note, not a finding: `Run` truncates the server log at start. D11 says only that the log opens with the pid. This is a small inference that the pid line must be the log's first line, and it is not a scope defect.

Tool calls: 0. My context already held everything this lens needed.
