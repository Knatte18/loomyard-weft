# An arras family filing turns the run's entry into its project and re-points its dependents

`arras-family-filing` · Task · feature, shed, loom, burler, arbiter, board · running · Worktree-Create

Decision record: [Sketch](decision-arras-family-filing.md)

Depends on: [`arras-recipe-perch`](design-arras-recipe-perch.md)

Filed from the Sketch `motif-raddle-recipe`.

Takes: raddle-perch-mode's `perch` mode and its project outcome whose slug equals the run's entry; the arras recipe of arras-recipe-perch; project-release-members' project release; Board.FileRaddle, checkRunLocks, endsAsSketch and sketchOf, Raddle-File's Call and finish, Board.SetRunStatus and battencli.ClearMotifRun
Delivers: Raddle-File's filing write turning the arras entry into a discussed project holding the whole record, with the part list body and `order`, admitted by the run lock only for that derived shape while the entry's status names Raddle-File; the restore of the entry's record on a failed write; the filing mark under shedrun.RunDir that makes loom's and batten's status writes skip the entry; the store-done and AlreadyFiled repair paths, the halt over a project not filed from the current proposal, and finish skipping EndEntry for a family; ClearMotifRun clearing a done arras run; every dependent's `depends_on` on the arras slug moved onto the filed parts no other part depends on, with a halt naming a refusing entry
Exports: the arras-project admission in boardengine's run lock, the filing mark and its reader in shedrun

The part changes these packages:

- internal/boardengine (about 220 changed lines)
- internal/loomshed (about 120 changed lines)
- internal/loomcli (about 60 changed lines)
- internal/battencli (about 80 changed lines)
- internal/shedrun (about 30 changed lines)
