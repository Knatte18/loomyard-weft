# Board: render one file per project, research notes first

The board rendered one `README.md` holding every project, task and note of the hub.
After the `audit` label became `research`, the research notes moved from first under Notes to last, under the heading `Researchs`.
This change renders one file per project, puts research first and names dependencies by handle.

## What changes

- `README.md` is the root project's file: the projects with no parent as linked bullets with their briefs, then every task and note that belongs to no project.
- Every project, open or done, renders to `project-<slug>.md` beside `README.md`.
  It opens with the project's handle and title, the render time, a link up to its parent's file, the brief, `beam` or `grouping`, and a beam's effective owner.
  It then holds the same sections as `README.md` for its own entries, and a nested project is listed in its parent's file.
- The split is rendering only: entries stay in the one store, and handles and layers stay hub-wide.
  A task in a project that waits on a root task stands in the same layer it did, and a project's file shows gaps in its handle numbering.
- The Projects section loses its lead line, its member table and its Standing column.
  A done project is a Done row of its parent's file, its title linked to its file.
- The After column names open dependencies by handle, one per line within the cell, in `depends_on` order.
  A dependency that names no board entry shows its slug as a code span.
- Notes run `Research` first, when `research` is a configured type, then the other types in `board.yaml` order, then Other.
  The heading is `Research`, not `Researchs`; a hub with no `research` type renders as before.
- `internal/fabricengine` replays a board commit that touches a `project-<slug>.md` file by entry, like `README.md`.
  Without it such a commit falls back to a git-level replay that conflicts on `board.json`, so every board write would break the records replay once project files exist.

`internal/boardengine/doc.go` and `internal/fabricengine/doc.go` describe the new layout.

## Design decisions

- The file name is `project-<slug>.md` with the prefix a constant in `render.go`, so no `board.yaml` key is added.
  The slug is unique and path-safe, and the render manifest drops the file of a removed project.
- `fabricengine` cannot import `boardengine`, which imports it, so `boardProjectPrefix` repeats `projectFilePrefix`; a comment on each names the other.
- `Render` stays pure, and the tests stay tier0 goldens in `render_test.go`.
- The replay change falls outside the original scope, which listed only `boardengine` files; it was added to the scope as forced by the new file shape.
- Links from outside the board to project sections of the old `README.md` break.
  No external anchor is known to rely on them.

## To check by hand

Nothing was run by hand.
A render against the real hub board is worth a look for the file set and links.
The first two items were open when the review ended; the others were addressed by later commits, which are worth confirming.

- `isBoardOverview` in `internal/fabricengine/replay.go` and its project-file handling are covered by a replay test, including rows where `project-.md`, `project-a/b.md` and `dir/project-x.md` are not overviews.
- The replay change in `internal/fabricengine/replay.go` is outside the render scope; confirm it belongs in this change.
- `TestRenderReadmeGolden` in `internal/boardengine/render_test.go` pins the After cell of a dependency that names no board entry.
- `TestRenderReadmeGolden` pins a draft task in a project file, as its own Draft subsection after the Ready and layered tasks.
- `boardProjectPrefix` in `internal/fabricengine/replay.go` and `projectFilePrefix` in `internal/boardengine/render.go` hold the same text.
