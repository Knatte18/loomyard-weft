# Mark a project's own entries apart from its sub-projects on the board

In the rendered `README.md`, Prime listed its open projects as `###` sections and then the root's own Tasks and Notes as `###` sections at the same level.
Nothing marked where the last project ended, so the root's own entries read as if they belonged to it.
A project file with sub-projects had the same problem one level up.

Now `projectBody` in `internal/boardengine/render.go` puts a project's own entries under a heading of their own whenever the project has open sub-projects:

- The heading sits after the sub-projects, at their heading level, with a meaning line under it, and the Tasks, Notes and Done sections move one level further down.
- For the root it is `No project`: "The entries under no project."
- For any other project it is `Own entries`: "The entries directly under this project and under none of its sub-projects."
- A project with no open sub-project renders as before, its entry sections directly under its heading.

Since `projectBody` serves both the README and every project file, both follow the change.
The heading-level cap at six still applies, so a deep chain of projects still renders.

The package doc in `internal/boardengine/doc.go`, the file header of `render.go` and the board paragraph in `docs/overview.md` describe the new heading.
The README golden test pins the `No project` heading in the README, `Own entries` in a nested project section and in a project file, and the unchanged layout of a project without sub-projects.

## Design decisions

- The two new headings carry no entry count, like `Prime`, since they group sections rather than list entries; `doc.go`'s rule that every section heading ends in its count now names them as exceptions.
- The headings are keyed on open sub-projects, as `childProjects` returns them, so a project whose only sub-projects are done keeps its entries directly under its heading.
