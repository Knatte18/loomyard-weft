```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I find what reads or asserts it and check whether the diff leaves any of it wrong.

**Per-change list**

1. **`BoardView.Projects` now includes every done project after the open ones** (`internal/boardengine/view.go:41-42`, `:190-194`). It is read in four places:
   - `board.html` `buildTree` (`internal/boardengine/board.html:402-421`): covered. Every node is created before any parent is linked, so the order of the appended entries does not matter. A done member now files under its done project, and P4 (open, under done P3) keeps `Parent: "P0"`.
   - `board.html` `projectSelect` (`internal/boardengine/board.html:583-598`, called from `:642` for the edit form and `:731` for the create form): **not covered**, see finding 1.
   - `boardserve` `namesFile` (`internal/boardserve/server.go:283-294`): covered. A done project's design file is now served by `/file`, and `renderDesigns` (`internal/boardengine/render.go:978-988`) writes a design file for every entry with a body, done projects included, so the file exists.
   - `TestRenderBoardData` `ProjectsListTheRootThenEachOpenThenEachDoneProjectWithItsParent` (`internal/boardengine/view_test.go:141-155`): updated in the diff.
2. **`viewFixture` gains T11** (`internal/boardengine/view_test.go:60`). Its users are `TestRenderBoardData` (whose entry map was updated with T11 at `:121`) and `TestViewerPage` (`internal/boardengine/viewer_test.go:27`), which asserts only the page bytes. Both are covered.
3. **Doc comments for `BoardView.Projects` and `viewProjects`.** The only other prose about the projection is `docs/overview.md:297`, which does not describe what the project list holds. Covered.
4. **Repository rules, PATTERN and commit trailers.** No PATTERN change is declared. The diff touches only `boardengine`'s view, which `PATTERN-code-conventions` and `PATTERN-tolerant-records` leave unaffected. The commit's `Root-Cause:` trailer names the defect the diff removes, so it is not just a symptom. Nothing outstanding.

**Findings**

1. MEDIUM · design · `internal/boardengine/view.go:190-194` (effect at `internal/boardengine/board.html:589`)
   - **Summary:** the page's project dropdown in both the create and the edit forms now lists every done project. Any write that picks one is always refused, because `checkProjectField` rejects a done project with "is done".
   - **Evidence:** `board.html:589` builds the dropdown from every project in the list (`state.view.projects.slice(1)`), with no done filter. `internal/boardengine/project.go:97-98` refuses the write (`case isDone(project): … "is done"`).
   - **Before the diff:** the list held only open projects, so the dropdown offered only valid targets.
   - **Fix:** either filter done projects out of `projectSelect`, or carry a status on `ViewProject` so the page can tell a done project apart. As it stands, a `ViewProject` has no field that marks it done, so the page cannot do this filtering.

**Tool calls**

1. A grep for readers of `Projects` and `"projects"`, and for docs about the projection: my context did not cover consumers outside `boardengine`.
2. Read `internal/boardserve/server.go:260-320` (`namesFile`) and grepped the docs: this is a reader of `view.Projects` that the manifest did not cover.
3. A grep in `render.go` for where design files are written, and in `board.html` for the dropdown: to confirm whether a done project's design file exists and how the dropdown is built.
4. Read `docs/overview.md`'s line on the projection and `renderDesigns` in `render.go`: to confirm the docs make no claim about the project list and that a done project's design file is written.
