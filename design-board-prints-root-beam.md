# The board prints the prime as the root beam `root`

`board-prints-root-beam` · Task · enhancement, board, orch · running · Drift-Verify

Decision record: [Sketch](decision-board-prints-root-beam.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: BriefTask.ParentBeam set in internal/boardengine/store.go from ParentBeam and hubgeom.ParentOf; the get envelope written by outputGetTask in internal/boardcli/cli.go; boardcli's --text listing; boardengine.RootSubject; the parent_beam readers in contracts/orchrole/orch-role.md and plugins/ly/skills/board/SKILL.md
Delivers: lym board list and lym board get printing parent_beam root for every entry under the prime; a P0 row among the projects in lym board list --text; boardengine.ParentBeam, Board.ParentBeamOf and hubgeom.ParentOf still returning the empty slug for the prime; the orch-role contract, the board skill and boardcli's help and package doc reading root as the prime and addressing its orch as <shortname>:orch
Exports: nothing new; the printed value of parent_beam changes at the printing boundary only

The part changes these packages:

- internal/boardengine (about 40 changed lines)
- internal/boardcli (about 60 changed lines)
- contracts/orchrole (about 15 changed lines)
- plugins/ly/skills/board (about 10 changed lines)
