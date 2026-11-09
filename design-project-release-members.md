# Releasing a project with lym board set-status releases its draft member tasks

`project-release-members` · Task · feature, shed, loom, burler, arbiter, board · running · Code-Burler-Fix

Decision record: [Sketch](decision-project-release-members.md)

Filed from the Sketch `motif-raddle-recipe`.

Takes: the board as it is today: Board.SetStatus and Store.SetStatus, validateWrite and the draft rule
Delivers: a null status from `lym board set-status` on a project with no status releases its direct draft member tasks in one write, validated as each member's own release, refusing as a whole on a failure and refusing a project with a run status or done with each member's release as the way forward; Board.SetStatus, Store.SetStatus, upsert and merge release no member
Exports: a Board method the set-status verb alone calls to release a project with its draft members

The part changes these packages:

- internal/boardengine (about 150 changed lines)
- internal/boardcli (about 80 changed lines)
