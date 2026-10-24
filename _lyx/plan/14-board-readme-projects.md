# Card 14 — board-readme-projects

**Edit:**
- `internal/boardengine#renderReadme`
- `internal/boardengine_test#TestRenderReadmeGolden`
- `plugins/ly/skills/board/SKILL.md`

**Create:**
- `internal/boardengine#renderProjects`

**Extends:**
- `internal/boardengine/doc.go`

**Intent:**
D15: the README gains `## Projects (n)` before `## Tasks`, n the number of open projects, with its meaning line ("Groups of entries; a beam spawns its members from its own worktree.").
Each open project is an H3 of its handle and title, then its brief, then `beam` or `grouping`, and for a beam its effective owner (the recorded `owner_machine`, else the prime owner), then one table of its members: the entries whose `project` names it, in the order of its `order`, then the members `order` leaves out by handle.
The member table's columns are the handle, the bold title, the kind, and where the member stands: its Running status (`atCell`), `Ready`, its layer, `Note` or `Done`.
Members keep their rows in the Tasks, Notes and Done sections, and a done project is a row of Done (card 9's render order); `renderProjects` writes the section, `renderReadme` calls it and leaves projects out of Tasks and Notes.
`TestRenderReadmeGolden` gains a golden with a beam project and a grouping project: ordered members, an unordered member after them by handle, a running member, a note member, a done member, the beam's owner, and a done project listed in Done.
The `ly:board` skill names the `project` kind, its `P` handle, the `project`, `order` and `beam` fields, that `beam` is set only by `lyx beam create` and `lyx beam promote`, and the `parent_beam` list field.

**ImpactSummary:** One new README section rendered before Tasks; the golden test is the only consumer of the README text.
