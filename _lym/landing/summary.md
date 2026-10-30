# Board README: a section per open project, Prime below, Prio column

A note that was a member of a project rendered only on that project's page and left the README's type groups, while the README listed the project as a single line with its brief.
The note looked gone, and a project with members looked as empty as one without.
The README now gives every open project its own section, so no entry of an open project leaves its view.

## What changes in the rendered board

- `README.md` opens, after its header and intro, with one `##` section per open project, nested projects included.
  The heading links the project's handle and title to its file and counts its open members.
  Under it come the brief, when there is one, and one table of its open members of every kind, laid out as a Notes type-group table with the Entry column in place of Note; an empty project shows `_None._`.
  This replaces the README's Projects bullet list.
  Project pages keep their Projects list of nested projects.
- The root project's own Tasks, Notes and Done sections sit under a `## Prime` heading below the project sections, with one line saying it holds the entries under no other project.
  Their headings drop one level (`### Tasks`, `#### Running`), and the Prime heading carries no count.
  Project pages keep their heading levels.
- The priority column of the Notes tables, and of the new project-section tables, is headed `Prio` instead of `P`, so it no longer shares a letter with a project's handle.
- A non-beam project page no longer shows the stray `grouping` line under its brief; a beam page still shows `beam` and its owner.

The change is in `internal/boardengine/render.go`, with its golden tests in `render_test.go`, the README description in `doc.go` and `docs/overview.md`.
The store, `lym board list` and every other view are untouched.

## Design decisions

- Project sections run depth-first: each project under the root by handle, each followed by the sections of the projects under it in its own order key, then by handle.
  The root has no order key, so its children sort by handle only.
- An open project under a done project counts as directly under the root, since a done project has no README section, and the project page still lists it.
  Without this, its open members would be missing from the README again.
- Done members stay out of project sections, as they do in the type groups; the project page's Done section lists them.
- The prime's section is its existing Tasks, Notes and Done content under a Prime heading, not a second table of the same entries.
- `renderBoardFile` is split into `renderReadme` and `renderProjectFile`, sharing `entrySections`, `table` and `boardIntro`, so the README and project pages differ only in what they put around the shared sections.
  Render stays a pure function, with section names and column headings declared in `render.go` alone.
- Demoting the prime's headings changes every README heading level.
  No Go code reads the rendered README, so only an external reader keyed on `## Tasks` would notice.

## To check by hand

Not run by hand; the golden tests in `internal/boardengine/render_test.go` cover the layout.
Open a rendered `README.md` on a board with nested projects and confirm the following.

- An open project under a done project has its own section, in `render.go` `childProjects`.
- Root-level project sections run by handle, as the README paragraph in `doc.go` and the `projectSections` comment say, rather than by the root's order key.
- A project-section member with a high or low priority and a body shows its `H` or `L` cell and a linked slug, in the project-section tables of `render_test.go`.
