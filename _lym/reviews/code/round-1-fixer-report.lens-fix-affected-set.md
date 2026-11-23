```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1 (project-choice sites): `state.view.projects` has two consumers, `buildTree` at `board.html:403` and `projectSelect` at `board.html:589`.
  - `projectSelect` serves both the create form and the edit form, and the fix filters inside it, so both are covered.
  - `buildTree` still needs the done projects to nest their members, so it is correctly unfiltered.
- F1 (`ViewProject` constructors): the only constructors are the root literal at `view.go:172`, where `Done` defaults to false, which is right for the root, and the `add` closure at `view.go:174`, which now takes `done`. Both of its call sites pass it: false from `walk` and true from the done loop.
- F2 (fixture consumers): I did not re-run this search, which was a coverage gap in my pass. The gate, run with P5 in `viewFixture`, passed all of tier 0 with 0 failures, so no other `viewFixture` consumer broke. Nested done projects keep their order after P3 in the test's expected rows.

Tool calls made:
1. `grep` for `view.projects`, `.projects` and `ViewProject{` across `internal` and `cmd`. It checked the sets F1 names (`projectSelect` consumers and `ViewProject` constructors), which my context did not show.
