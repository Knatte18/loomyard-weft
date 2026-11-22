# Orch procedure to move a running task by a move note and a fresh run

`orch-move-running-task` · Task · feature, orch, heddle · running · Drift-Expect

Decision record: [Sketch](decision-orch-move-running-task.md)

Filed from the Sketch `move-task-by-handback`.

Takes: the Sketch's decision record; the existing verbs lym creel post, lym tender pause and resume, lym batten cancel --remove, lym board get, upsert and set-status; role-beams.md and orch-role.md as per-task-settings-one-lookup leaves them
Delivers: a section of contracts/orchrole/role-beams.md holding the move procedure of D1 to D7; a pointer to it beside the cancel lines of orch-role.md's run loop and in its line naming what role-beams.md covers; the move note line in internal/heddleengine/glossary.md (D8)
Exports: the term move note and the role-beams.md section on moving a running task to another parent

The part changes these packages:

- contracts/orchrole (about 90 changed lines)
- internal/heddleengine (about 1 changed lines)
