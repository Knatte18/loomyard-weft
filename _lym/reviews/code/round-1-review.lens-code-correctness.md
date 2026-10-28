```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on concrete inputs.

**Per function and case**
- `Render` (render.go:88-103). Input: a project `ship` plus an empty `Outputs.Readme` case is not run. Input: `[]Task{}` returns only `README.md`, and one project returns `README.md` and `project-<slug>.md`. Correct.
- `fileHeader`, root. Input: the zero Task. It returns the title `Board`, the render time and the prime owner, as before. Correct.
- `fileHeader`, a root-level project. Input: `Project == ""`. `b.bySlug[""]` gives the zero `TaskWithLayer`, so `fileLink` takes the root branch and links `[Board](README.md)`. Correct.
- `fileHeader`, a nested project. Input: a child of `ship`. It links `project-ship.md`. A done parent still has a file, so the link resolves. Correct.
- `fileHeader`, a project whose `Project` names a slug that is not on the board. The lookup gives the zero Task, so the `Under:` link points at README.md, which does not list it. Not a defect; see F1.
- `fileHeader`, `Beam` branch. Input: a beam with no recorded owner. `ownerOf` goes through `ParentBeam` and falls back to the prime owner, so the output is `Owner: <prime>`. Correct.
- `fileHeader`, `!Beam` branch. It returns `grouping`. Correct.
- `renderBoardFile` filter. Input: `twl.Project == project.Slug`. For the root this is the empty string, so only entries with no project match. Correct. The heading counts are computed from `own`, so they count only this file's entries.
- `renderBoardFile` layers. `RenderOrder` runs once over the whole board, so layers stay hub-wide (acceptance D4). A task in project `ship` waiting on a root task keeps the layer it had. Correct.
- `renderBoardFile`, Done table, project row. A done project renders `**[Title](project-slug.md)**` and drops its brief and labels, which matches D3. A title containing `]` is not escaped in the link text. That is cosmetic and I raise no finding.
- `projectsSection`. Input: `own`, filtered to open projects. The sort is `position` (index in `project.Order`, else `len(Order)`), then `compareHandles`, then ID. For the root, the zero Task has a nil `Order`, so every position is 0 and the sort is by handle. This matches D3. In the golden, `ship.Order = ["later"]` gives `P5 Later` before `P4 Inner`. Correct.
- `projectsSection`, empty. It returns nil when there are no open children, so the section is omitted. Correct.
- `projectsSection`, brief. A brief with a newline goes through `cellText`, which makes it a space. A `|` in a brief renders as `\|`, which GFM accepts outside a table. Correct.
- `afterCell`. Dependency states:
  - Open with a handle: the handle. Correct.
  - Done: skipped. Correct.
  - Unknown slug: a code span. Correct.
  - Open with an empty handle: a code span of the slug. Correct.
  - Empty `DependsOn`: `strings.Join(nil, ...)` gives `""`, a blank cell. Correct.
  - Single dependency: no break. Correct.
  - Several dependencies: joined with `<br>` in `depends_on` order. Correct.
- `notesTypeOrder`. Input `[bug, enhancement, research]` gives `[research, bug, enhancement, ""]`. Input with no `research` type gives `[...types, ""]`, the old order. Input `nil` gives `[""]`. Duplicate `research` entries in `types` collapse to one. Correct.
- `typeHeading`. `research` gives `Research`, `bug` gives `Bugs`, and `""` gives `s`, as before. Correct.
- `newBoardView` and `ownerOf`. `readmeOwners` is now built once per render instead of once per section. Correct.
- `isBoardOverview` (fabricengine/replay.go:524).
  - `README.md` is true.
  - `project-x.md` is true.
  - `project-.md` is false because the slug is empty.
  - `project-x.txt` is false.
  - `dir/project-x.md` is false: the prefix does not match, so `found` is false.
  - `project-a/b.md` is false through the `/` check.
  - `design-project-x.md` is false.
  - Correct.
- `localBoardEntries` filter (replay.go:637). A project file is now both a board path and an overview, so it is excluded from entry paths. Correct.

**Findings**
- **F1: LOW, design.** `render.go:271-274` and `render.go:247`. An entry whose `Project` names a slug that is not on the board appears in no file: it is filtered out of README.md (its project is non-empty) and no project file exists for it.
  - Before this change such an entry still rendered, in the one README.
  - `ParentBeam` (`parentbeam.go:23-26`) tolerates a missing project, and a hand-edited `board.json` can produce one. A project chain that loops back on itself, which `parentbeam.go` says a hand edit can store, likewise leaves its entries unreachable from README.md.
  - A write cannot store a dangling project, since prune keeps a done project while a member names it. The impact is therefore confined to hand edits, which is why I rate it LOW.
  - Fix, if wanted: render entries whose project is not a board entry in README.md.

I found no wrong result, swallowed error or inconsistent state on a failure path in the changed code. `Render` and `renderBoardFile` still return errors only from `RenderOrder`, and that error propagates.

**Tool calls (3)**
- `grep boardOverviewFile|isBoardOverview internal/fabricengine/*.go`: to see every use of the overview-file test. My context did not show them.
- `grep dangling/unknown project in internal/boardengine/*.go`: to see whether a write validates that a project exists.
- `sed parentbeam.go` and `layer.go` RenderOrder: to read the `ParentBeam` tolerance for missing projects and loops.
