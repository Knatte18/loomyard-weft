# Support log — board-readme-per-project

## Interview

A darn run holds no interview; the board entry's body is the whole statement of the change.

## Rejected alternatives

- A `board.yaml` key for the project file prefix: adds a template unit and a reconcile for no asked need.
- Files only for open projects: a done project's members, kept while they name it, would have no file.
- Reordering the hub's board.yaml types instead of the renderer: the hub config lives outside this repository, and the heading "Researchs" still needs the renderer.
- Keeping the member table in the parent file beside the per-project file: renders every member twice.
- Linking After handles to their files: a handle already names one entry, and links add width to the cell.

## Review rounds

_No rounds yet._

## Question ledger

- Where does a done project's file go? Self-pick: every project gets a file (D1).
- Does a project file keep the old member order? Self-pick: the order key orders child projects in the Projects list; tasks and notes follow the layer and handle order (D3, D4).
- Are layers per file or hub-wide? Self-pick: hub-wide (D4).
- How is a dependency shown when it names no entry? Self-pick: its slug as a code span (D5).
