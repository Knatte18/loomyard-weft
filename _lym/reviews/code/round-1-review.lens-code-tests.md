```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per-behaviour list**

1. `consoletitle.Title` joins the parts, lowercases the shortname and drops empty parts. `TestTitle` pins it (`consoletitle_test.go:345`). It has rows for every part, no project, a label after a project, an upper-case shortname and no shortname.
2. `consoletitle.Workspace` adds the suffix and drops an empty shortname. `TestWorkspace` pins it (`consoletitle_test.go:369`).
3. The leaf import cap on `consoletitle`. `TestLeafInvariant_StandardLibraryOnly` pins it (`leaf_enforcement_test.go:421`).
4. `boardengine.worktreeWindowTitle`, through `IDEWindowTitle` and `TerminalWindowTitle`. `TestWindowTitles` (`handles_test.go:221`) pins these cases, each for both titles:
   - prime
   - beam
   - task in a project
   - task in no project
   - task under a beam
   - task in a project under a beam, which takes the beam
   - slug off the board
   - no shortname
   - unreadable board
   - no board with the prime
5. The case "a task whose beam is closed (`done`) takes its `project`'s handle". **None.** `ParentBeam` skips a done beam, and `worktreeWindowTitle` then falls back to `entry.Project`. `TestWindowTitles` has no done beam. The focus directive names this case, and the `ideengine` fixture's done beam `gone` has no task under it.
6. The case "a task whose `project` slug is missing from the board gets no `<P#>`". **None.**
7. The case "primeName is empty". **None.** `TestWindowTitles` always passes `prime = "main"`. The doc comments on `lookFor` and `terminalWindowTitle` promise that an unresolvable prime titles the worktree as a task, and no row exercises that.
8. `ideengine.lookFor` takes its title from `IDEWindowTitle`. `TestLookFor` (`look_test.go:84`) pins it for the prime, beams, tasks and slugs off the board. The `lymcwd` integration tests pin the lower-cased shortname. This is covered, and the title rows partly duplicate `TestWindowTitles`, but each row also pins colours, so they stay.
9. `fabricengine.workspaceWindowTitle` uses `consoletitle.Workspace`. `TestRegenerateHubWorkspace` (`hubworkspace_integration_test.go:360`) and the `ideengine` spawn integration tests pin the lower-case workspace titles for both the hub and a beam.
10. `reedcli` window verb: the default title comes from the `windowTitle` seam, and `--title` wins. `TestWindowCmd_OpensATerminalAttachedToTheToldWorktree` pins both (`window_test.go:44`, with the `TitleFlag` and `DefaultTitleIsTheFormedTitle` rows). The seam asserts that the directory passed is the told one.
11. `reedcli.terminalWindowTitle` itself has three behaviours:
    - fall back to the base name when the directory is in no worktree;
    - warn and title as a task when the prime is unresolvable;
    - resolve the worktree and prime, then call `TerminalWindowTitle`.

    **None.** The seam bypasses the function in every row, and no other test reaches it.
12. The reed templates drop `--title {slug}` for the task, beam and prime button groups. `TestLoadConfig_TemplateDefaultsResolve` pins the three groups (`config_test.go:95-115`). The static template text itself is exempt.
13. batten's `{title}` placeholder now calls `TerminalWindowTitle`. `TestCommandDeps_TitleOpensTheSlugWithTheTasksHandle` (`commanddeps_test.go:17`) reaches it, but only with no shortname recorded and no project. Its rows pin `T4: run` and the bare slug, so none would fail if the call lost the shortname or project handle.
    - Those parts are pinned only via `TestWindowTitles`, which is enough for the join.
    - The test's name and its second comment line still describe the old "slug opened with the handle" behaviour.

**Findings**

- **LOW / design** — `internal/boardengine/handles_test.go:221`. `TestWindowTitles` has no row for a task whose beam is `done` (it should show the project's handle, via the `entry.Project` fallback after `ParentBeam` skips the beam). It has none for a task whose project is missing from the board, or for an empty `primeName`. These are the branches `worktreeWindowTitle` adds beyond `ParentBeam`, and the focus directive names the closed and missing case. Add rows to the existing table. Evidence: `handles.go` `worktreeWindowTitle` fallback `project = entry.Project`; `parentbeam.go:32`.
- **MEDIUM / design** — `internal/reedcli/window.go:689` (`terminalWindowTitle`). The new function has no test: the `windowTitle` seam replaces it in every row of `window_test.go`. The base-name fallback, the unresolvable-prime Warn path and the resolve-then-title path are uncovered. Evidence: `window_test.go:62-69` always sets `windowTitle`; nothing else calls `terminalWindowTitle`.
- **NIT / consistency** — `internal/battencli/commanddeps_test.go:15-17`. The test name and comment still say the title is "the slug opened with the handle". The rows do not show that `Title` now forms the full console title with the hub shortname and project handle. Rename it, or add one row with a recorded shortname and a project.

**Tool calls:** none. The diff and definitions already in my context were enough.
