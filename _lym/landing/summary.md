# The board prints the prime as the root beam `root`

`lym board list` and `lym board get` now print `parent_beam` as `root` for every entry whose nearest beam is the prime, where they printed an empty value or left the key out.
An empty value read as "no beam", while the board view and window titles show the prime as `P0`.
`root` is `boardengine.RootSubject`, the slug `lym adopt root` and `lym beam stop root` already take.

## What changes

- `internal/boardengine/store.go`: `Store.ListTasksBrief` sets `BriefTask.ParentBeam` to `RootSubject` when the entry resolves to the prime.
  A failing `Config.ParentOf` seam still leaves the value empty, so `root` never stands in for a parent the board could not read.
- `internal/boardcli/cli.go`: `outputGetTask` prints the prime as `root` in the `get` envelope.
- `lym board list --text` shows the root project as a `P0` row with slug `root` and the title the board files give it, `Root: entries under no project`.
  The row comes from the new `boardengine.RootProjectBrief`, built from the same `rootProject` value the README renders, and `boardcli`'s `withRootProjectRow` places it before the first project or note, which in README order puts it ahead of the projects.
  The JSON `tasks` array gets no root row.
- The readers of the printed value move with it: `contracts/orchrole/orch-role.md` (which entries the prime's orch reads and releases, and its parent orch address `<shortname>:orch` for `parent_beam` `root`), `plugins/ly/skills/board/SKILL.md`, and `boardcli`'s help and package doc.

## Design decisions

- The mapping sits at the printing boundary only.
  `boardengine.ParentBeam`, `Board.ParentBeamOf` and `hubgeom.ParentOf` keep returning the empty slug for the prime, since every owner read and beam verb takes it as the prime; a new assertion in `TestParentBeam` pins `ParentBeamOf` at the empty slug.
- `list --text --label …` leaves the `P0` row out: a label filter keeps only entries carrying the label, and the root carries none.
- The `P0` row has no dates, labels or owner, since the root is no board entry; the prime's owner stays on the listing's `prime owner:` line.

## Tests

- `TestParentBeam`, the store listing test and `stepProjects` expect `root` where they expected an empty or absent `parent_beam`.
- `stepListAndFindText` expects the `P0` row between the open task and the note, with the columns widened to it; `find --text` shows no root row.
