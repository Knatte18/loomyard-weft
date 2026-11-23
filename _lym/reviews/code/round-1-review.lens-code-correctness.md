```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over `viewProjects` in `internal/boardengine/view.go` and the data it hands `board.html`.

**Cases run**

- **`add` closure** (`view.go:171-181`)
  - Case: open project P1 from the fixture, which has a body, a brief and labels.
  - Every field comes from the same `Task` the old inline literal read, and `b.designFile(project)` gets the same value `child.Task` passed before.
  - P1 gives `design-alpha.md` and P2 gives `""`, with labels made non-nil.
  - Correct.
- **`walk` over open projects** (`view.go:182-189`)
  - Case: the fixture. Its open projects are P1, then P2 under P1, and P4, which sits under done P3 and so counts as under the root.
  - The order and parents are the same as at the base, because `walk` was only refactored onto `add`.
  - Correct.
- **Done-project loop: no done project** (`view.go:190-194`)
  - Case: an empty board, where `b.ordered` is empty, and a board with only open projects.
  - Nothing is appended, so the output is the root alone, or the same as at the base.
  - Correct.
- **Done-project loop: done project with no `Project`**
  - Case: P3 gamma.
  - `projectHandle` misses in `bySlug[""]` and returns the root's handle, P0.
  - P3 is listed once, after every open project, with parent P0.
  - Correct.
- **Done project under an open project**
  - Case: done X with `Project: "alpha"`.
  - `bySlug["alpha"]` is found with handle P1, so the parent is P1.
  - P1's Done section lists X (`render.go:685-696`, which uses `twl.Project == project.Slug`, done entries included).
  - Correct.
- **Done project under a done project**
  - Case: done Y with `Project: "gamma"`.
  - Its parent is P3, which is itself appended by the same loop.
  - `buildTree` creates every node before it links any parent (`board.html:406-411`), so it does not matter which of the two comes first.
  - Correct.
- **Done project whose `Project` names a slug not on the board**
  - `projectHandle` falls back to P0. This matches `upLink`, which also links such a project's file up to the root.
  - Correct.
- **Listing a project twice**
  - `childProjects` keeps only `!isDone` projects (`render.go:721`), so `walk` never reaches a done project. The loop takes only done projects.
  - The two sets are disjoint, so no handle is listed twice.
  - Correct.
- **`buildTree` and `projectsView` with done projects listed**
  - The members of a done project now attach to its node instead of falling back to the root (`board.html:413`).
  - A done project with no member left after the filters has `total === 0` and is hidden (`board.html:434`).
  - Correct.
- **`projectSelect` reading the widened `projects` list** (`board.html:589`): **not correct.** See finding 1.

**Findings**

1. **Severity MEDIUM, class `design`**
   - Location: `internal/boardengine/view.go:190-194`, as it reaches `internal/boardengine/board.html:589`.
   - Summary: the viewer's project picker, in both the create and the edit form (`board.html:642`, `board.html:731`), builds its options from `projects.slice(1)`. Every done project now appears there, but the write path refuses an entry whose project is done (`internal/boardengine/project.go:97-98`, `checkProjectField`, the case `isDone(project)` with "is done"). The page offers a choice that always fails.
   - Evidence: `board.html:589-591` adds an option for every entry of `state.view.projects` after the root; `project.go:97-98` refuses a done project.
   - Suggested fix: skip done projects in `projectSelect`. The `ViewProject` entry would need something that marks it done, such as a status field, because it carries no status today.

No tool calls were made.
