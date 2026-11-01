# Board marks a project's own entries apart from its sub-projects

In the rendered `README.md`, the Prime section listed its projects as headings and then the root's own Tasks, Notes and Done at the same level.
Nothing marked where the last project ended, so the root's entries read as that project's.

A project with open sub-projects now puts its own Tasks, Notes and Done under one heading of their own.
The heading sits after the sub-projects at their level, with the entry sections one level further down, and a meaning line under it.
For the root the heading is `No project`; for any other project it is `Own entries`.
A project with no open sub-project renders as before, its entry sections directly under its heading.

The change is in `projectBody` in `internal/boardengine/render.go`, so the README and every project file follow it alike.
The package doc in `internal/boardengine/doc.go` and `docs/overview.md` describe the new layout.

Closes: none; this resolves the board entry `board-no-project-heading`, which names no GitHub issue.

## Design decisions

- The two headings and their meaning lines are declared beside the other fixed sections in `render.go` (`noProjectSection`, `ownEntriesSection`).
- Neither heading carries an entry count, unlike Tasks, Notes and Done, since it groups sections and lists no table itself; `doc.go` names them as exceptions to the count rule.
- Heading levels still stop at six, so a deeply nested project's `Own entries` heading and its entry sections can share level six.
  The golden in `render_test.go` pins this case.

## To check

- `render_test.go` goldens were rewritten for the new levels; read the nested-depth case, where headings stop at level six, by eye.
- Not run by hand: `lym board` against a real board to look at the rendered `README.md` and a project file with sub-projects.
