```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing the diff's changes to the record's decisions and added entries.

**Changes → decision**

| Change | Traces to |
|---|---|
| `view.go` projection and `board-data.js`; `renderFiles`; `unmetDependencies`; `Outputs.Labels`; `migrateLoaded`; `Board.View` (card 1) | D1, D8 |
| `EntryRevision`, `BoardRevision` | D8, D10 |
| `expect.go`; revision check in `boardCriticalSection` | D8 |
| `board.html`, `viewer.go`, LF pin in `.gitattributes` | D2, D4, D10, D11 retry (the LF pin keeps D2's bytes stable) |
| `FabricReplay.Claims` for both files | D3 |
| `boardserve` handler, writes and checks | D9, D10, D13 |
| `boardserve` lifecycle, state, launcher, watch loop; `lymdirs.BoardServeDirName`; exported `EntryStoreFile` | D9, D11 (`EntryStoreFile` forced by the watch loop) |
| Long poll keyed on `after` being present; page paces an immediate "unchanged"; one-second floor on event wakes | forced by D10/D11 and the one-second-floor constraint |
| `cmd/lym` daemon-wakeups list and `PATTERN-daemon-wakeups` background | forced by D11 (record constraint) |
| `lym board open` and `serve`, opener files | D5, D11, D13, added entry "Opener waited on through cmd.Wait" |
| `IsRunAgentSession` export and its callers | D12 |
| `runproc.RoleBoardServer`, its doc and test row | D11 |
| `rerender` Short reworded | forced by D1/D2 |
| `board open` task, two keys, `vscode` helper refactor, ide test swap | D6 |
| Workspace-content seam, build-all-before-write, `BuildWorkspace` members | D6, added entry "workspace seam" |
| `WorkspaceTasks`, `ideengine.WorkspaceContent`, `main.go` install, prerun seam test | D6 |
| `docs/overview.md`, package docs | forced by the above |

**Decisions → changes**

| Decision | Built by |
|---|---|
| D1 | projection and `board-data.js` |
| D2 | embedded page |
| D3 | render, manifest and replay claims |
| D4 | `board.html` |
| D5 | `open` verb and opener |
| D6 | tasks, keys, seam, `beam start` member |
| D8 | revision functions and `Expecting` |
| D9 | server checks, state, launcher |
| D10 | routes, `write.go`, page forms and actions |
| D11 | lifecycle, spawn, `runproc` role |
| D12 | run-agent refusal, emptied strand name |
| D13 | package split |
| Added: per-body folds moot; phone goal met | nothing to build; README layout unchanged |
| Added: no `launch.json`; prime open chain and `LY:` title stay out | nothing to build; neither appears in the diff |
| Added: workspace seam | built as stated (mark refusal in fabric; `folders`/`settings` refusal and splice in `vscode`) |
| Added: board files stay committed outputs | D1–D3 changes |
| Added: `open` refusal dropped; rerender line on the `file://` page | `open` reads neither file; the page shows the line |
| Added: parent confirms the two above | same changes |
| Added: opener through `cmd.Wait` | `startOpener` |

**Findings:** none. No change lacks a decision or a forcing change, and no decision that asks for a change is left unbuilt. Nothing on the record's `Out` list appears in the diff. I judged whether each decision has a build, not whether each build is correct.

**Tool calls**

1. `grep` for `rootProject` and `RootProjectBrief` in `boardengine`: D1 lists a brief for the root project, and the definition was not in my context. The root has no brief, so an empty one is correct.
