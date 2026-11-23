```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, which finds the test row that pins each behaviour the diff adds or changes.

**Per behaviour, the test that pins it:**

1. **`viewProjects` lists a done project after the open ones.**
   Pinned by `TestRenderBoardData/ProjectsListTheRootThenEachOpenThenEachDoneProjectWithItsParent` (`internal/boardengine/view_test.go:141-155`).
   Its `reflect.DeepEqual` fails if P3 is missing or placed before P4.
2. **Only done entries of kind project are added; done tasks T7 and T11 are not.**
   Pinned by the same subtest: an extra entry breaks `DeepEqual`.
3. **Open projects are not listed twice, because the `isDone` filter keeps them out of the appended loop.**
   Pinned by the same subtest.
4. **A done project's `Parent` is the project whose Done section lists it, through `b.projectHandle(twl.Task)`.**
   Not pinned. The fixture's only done project, P3 (`view_test.go:45`), has no `Project`, so `projectHandle` falls back to the root handle.
   A mutant that hard-codes `rootProject.Handle` at `view.go:192` passes every row.
5. **Done projects come in README order among themselves.**
   Not pinned. The fixture holds one done project, so no row checks the order of two.
6. **The `add` closure refactor carries the same Handle, Slug, Title, Brief, Labels, Parent and DesignFile for open projects.**
   Pinned by the P1 row, which has Brief, DesignFile and Labels (`view_test.go:145`), and the P2 and P4 rows (`view_test.go:146-148`).
7. **A done member of a done project is placed under that project (T11 → P3/Done).**
   Pinned by `TestRenderBoardData/EntriesStandInTheirProjectAndSection` (`view_test.go:121`).
   The entry's `Project` comes from the unchanged `projectHandle`, so this row pins existing behaviour, not the diff's.

**Findings:**

- **MEDIUM · design · `internal/boardengine/view_test.go:141-155` (row missing for `view.go:190-194`).**
  No test pins a done project's parent when that parent is not the root. That is the case the doc comment states at `view.go:167-168` ("a done one the project whose Done section lists it") and the one the commit message claims.
  The fixture's done project P3 has no `Project` (`view_test.go:45`), so its expected `Parent: "P0"` (`view_test.go:150`) matches the fallback branch of `projectHandle` (`view.go:247-250`). Replacing the call at `view.go:192` with `rootProject.Handle` keeps every test green.
  Fix: add a done project to the fixture whose `Project` names another project, ideally another done one such as P3, so the nested-beam case is covered. Expect its row with that project's handle as `Parent`, after P3. The same row then pins the README order among done projects (behaviour 5).

Tool calls: none.
