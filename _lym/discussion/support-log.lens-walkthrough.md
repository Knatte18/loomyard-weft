```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case and checked the end state against the decision's purpose.

**Findings**

1. **BLOCKING — D6: `alt+shift+0` in a beam window is ambiguous and can start the wrong beam.**
   - The hub workspace and each beam workspace are multi-root windows.
   - The beam workspace for `adopt-build-resplit` lists the folders `adopt-build-resplit`, `main` (the prime, path `loomyard`), `Bolt` and every task pair folder.
   - D6 writes a `beam start` task into both the prime's and the beam's `.vscode/tasks.json`. Both folders sit in that window, so two tasks carry the label `beam start`.
   - `workbench.action.tasks.runTask "beam start"` then either asks the operator to pick or runs the prime folder's task. That task runs `lym beam start` with no slug in the prime's directory, which brings up the root instead of the beam the window belongs to.
   - `board open` is written into every folder, so it is duplicated across all of them. Its copies are harmless because each resolves the same hub, but the key may still raise a picker.
   - The record's only stated bound is the task-pair window; it misses this case.
   - Possible fixes: put the tasks in each `.code-workspace` file's `tasks` key, so the window holds one copy; or give the command an explicit slug (`lym beam start root` or `lym beam start <beam>`) and give each task a label unique to its window.
   - Evidence: `/home/knatte/Code/loomyard-LYMHUB/loomyard-adopt-build-resplit.code-workspace`, and `ideengine.seed`, which sets `launchPath` to the workspace file for the prime and for beams.

2. **MEDIUM — D1: a project entry has no section, and the record does not say how projects appear in the data.**
   - Case: project P4 `adopt-build-resplit`, which has its own Running, Ready, Tasks, Notes and Done.
   - D1 says each entry carries "the README section the entry sits in", but a project is a grouping, not a row in any section.
   - D4 groups by project and also offers a "project" kind filter.
   - The record never says whether projects are entries with an empty section, or a separate top-level list carrying their own handle, brief and parent project.
   - Sub-projects nest under a parent project, as in "a project with open sub-projects puts its own Tasks… under `Own entries`". D4's "by project, then by section" grouping does not say how that nesting or the `Own entries` heading maps across.
   - The viewer would be handed data its grouping step cannot place.

3. **LOW — D5: "takes no argument and opens only that one file" does not hold.**
   - The board subtree has a `--board-path` flag (`internal/boardcli/cli.go` mentions runs "under --board-path").
   - So `lym board open --board-path X` would open `X/board.html`.
   - Fix: state the flag in the bound, or refuse it for `open`.

4. **LOW — D5: "both openers hand the file to the desktop and return" is not always true.**
   - With no desktop environment detected, `xdg-open` falls back to running `$BROWSER` or a text browser in the foreground.
   - The verb then blocks the VS Code task until the browser exits.
   - This is harmless in a task terminal, but the claim should be softened.

**Decisions walked**

- **D1** — Entry T21: Render produces `window.BOARD`; its section comes from `layerSection` and its labels are split by the vocabulary. Met for tasks and notes. Not met for projects (finding 2).
- **D2** — An identical-bytes rewrite through `fsx.AtomicWrite` leaves git clean. After an upgrade, the next write's sync commits the new viewer. Met.
- **D3** — The boardengine replay `Apply` (in `internal/boardengine/replay.go`) ends with `RenderToDisk`. Once both names are claimed, a replay re-renders them, and the manifest's stale-file removal covers them. Met.
- **D4** — A body holding `</script>` is escaped by the JSON encoding (D1), escaped again before the Markdown subset runs, then rendered. A missing `board-data.js` leaves `window.BOARD` undefined, and the page shows the rerender line. Met.
- **D5** — Run from the task pair `board-viewer`: the board dir comes through lymcwd, then `<hub>/_bolt/board.html`, then `xdg-open`. Met, subject to findings 3 and 4.
- **D6** — Seeding a task pair writes `reed attach` and `board open`: met. In a beam window, `alt+shift+0` is ambiguous: not met (finding 1).
- **D7** — Project P1's section becomes a Tasks block open by default and a Notes block closed by default, inside the project's `<details open>`. Met.

**Tool calls**

- `grep` of `internal/vscode/workspace.go` and `hubworkspace.go`, plus `ls` and `head` of the hub `.code-workspace` files: I needed to know which folders a prime or beam window holds, which my context did not show.
- `sed` of `internal/boardengine/replay.go` lines 440–480: I needed to confirm the replay re-renders after applying. My context showed only `Claims`.
- `grep` of `internal/boardcli/cli.go` lines 60–130: I needed to know how the board verbs resolve the board dir, and whether a path flag exists.
