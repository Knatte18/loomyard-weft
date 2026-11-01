```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it. Every row below is in `TestRenderReadmeGolden` in `internal/boardengine/render_test.go`.

**Per behaviour, the test that pins it**
1. The root section is last in README.md as `## [P0 Root: entries under no project](project-root.md) (n)`. It is pinned by the `rootProjectHeading` calls in every README row, and the projects row has the root after P6.
2. The facts line is `Owner <prime owner>` with no slug, age or design link. The `neve` rows pin the owner case and the `none recorded` rows pin the empty case.
3. The root's member count leaves out open sub-projects and done entries. The projects row pins it with `rootProjectHeading(2, 1, ...)`, where the root has one open task, one done project and several open projects. Counting the projects would change the number.
4. Top-level projects render as `##`, sub-projects nest one level per depth, and the `Prime` wrapper is gone. The projects row and the lone/loose row pin it.
5. An open project under a done project is placed at top level, before the root. The P6 orphan in the projects row pins it.
6. The root's own entries sit directly at level 3 in README.md, with no `Own entries` or `No project` heading, when the root has open sub-projects. `rootEntriesWithProjects(3)` in the projects row pins it.
7. `project-root.md` has the header, `Under: [Board](README.md)`, the facts line, the intro and the own entries at level 2. The `files` entry for `project-root.md` in the projects row pins it. `projectFiles` also lists it, so the key must be present in the rendered map.
8. The six-level cap still holds. The depth-six row gained P5, so the Own entries chain and the cap are pinned.
9. Empty sections for the root with no entries or projects are pinned by the lone/loose row (`rootProjectHeading(2, 0, ...)` plus `emptyProjectEntries(3)`) and the depth row.
10. `upLink` sends a project whose parent is not on the board to README.md, and a project with a parent to its parent's file. The `Under:` lines of the project files in the projects row pin both. I did not re-read those lines to confirm the orphan's file (`project-orphan.md`) is among the asserted ones.

**Findings**
- **LOW, design**, `internal/boardengine/render.go:163` and `render.go:105`:
  - `rootProjectFile` is `projectFileName("root")`, and the loop at `render.go:107-111` writes `result[projectFileName(twl.Slug)]` after the root's entry. A real project with slug `root` would overwrite `project-root.md`.
  - No test row covers this, and no code guards it. The slug validation I saw (`validateSlugLength`) does not reserve `root`.
  - Fix: add a row, or reserve the slug, so the collision is either rejected or its outcome pinned. This overlaps the correctness lens.
- **NIT, consistency**, `internal/boardengine/render_test.go`:
  - `project-root.md` is asserted in one row only, the one with projects.
  - The root file with no sub-projects, or a done-only root, has no row. Its body path is the same `entrySections(project, level+1)` call, which the README rows cover, so this is minor.
  - No separate replayer test pins `isBoardOverview("project-root.md")`. The prefix logic in `replay.go:66-73` is generic, so I raise no finding.

**Tool calls:** none. The manifest already held the test diff and `render.go`, so I read nothing further.
