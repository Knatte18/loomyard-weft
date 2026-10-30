# Board README: a section per project, Prime below them, a Prio column

The board README listed each project as one bullet with its brief, so a project's members showed only on its own `project-<slug>.md` page.
A note filed under a project looked gone from the README, and a project with members looked as empty as one without.

## What changes

- **Project sections.**
  After its header and intro, `README.md` holds one `##` section per open project on the board, nested projects included.
  The heading links the project's handle and title to its page and counts its open members.
  Under it come the brief and one table of the open members of every kind, laid out as a Notes type table (`#`, `Prio`, `Entry`, `Slug`, `Age<br>Revised`, `Labels`), or `_None._` when there are none.
  Sections run depth-first: the root's projects in the root's order key and then by handle, each followed by its own nested projects in the same order.
  The README's old Projects bullet list is gone; project pages keep theirs.
- **Prime section.**
  The root's own Tasks, Notes and Done move under a `## Prime` heading with a one-line meaning, below the project sections, each one heading level deeper (`### Tasks`, `#### Running`).
  Project pages keep their heading levels.
- **Prio column.**
  The priority column of the Notes tables, and of the new project tables, is headed `Prio` so it no longer shares the letter `P` with project handles.
- **No `grouping` line.**
  A non-beam project page shows its brief and nothing after it; a beam page still shows `beam` and its owner.

In `internal/boardengine/render.go`, the shared `renderBoardFile` splits into `renderReadme` and `renderProjectFile` over common `entrySections` (with a heading level), `table` and `childProjects` helpers; `projectSections` builds the new README sections.
The package doc in `internal/boardengine/doc.go` and the board entry in `docs/overview.md` describe the new layout.

## Design decisions

- Nested projects get their own README section too, since the reported failure was an entry hidden from the README; the sections are flat `##` in depth-first order rather than nested heading levels, so a deep project never sinks below the type-group levels.
- The prime's section is its existing Tasks, Notes and Done content under a Prime heading, not a second table of the same entries, which would list every root entry twice.
- Project sections list open members only, as the type groups do; a project's done members stay in its page's Done section.

## Tests

`TestRenderReadmeGolden` in `internal/boardengine/render_test.go` now pins the Prime heading and demoted levels on every README row, the `Prio` header on every Notes table, project sections with a linked counted heading, brief, member table and `_None._`, their depth-first order by order key then handle, and project pages without the `grouping` line.
