# Merge main into the beam adopt-build-resplit

`adopt-build-resplit-merge-in` · Task · feature, enhancement, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut

Merge in from parent: bring the parent branch main into the beam adopt-build-resplit.
This task's branch forks from the beam, so its landing carries the merge into the beam, as a true merge with no pull request.
Filed by `lym beam merge-parent adopt-build-resplit`.

- Run `lym fabric merge-in main` in this task's worktree.
  On a conflict, settle each path it lists, stage them with `lym fabric merge-stage <paths>` and conclude with `lym fabric merge --continue`.
- Make the gate pass on the merged tree: fix what the merge broke, on either side, and nothing else.
- Make no change the merge does not need: no refactor, no feature and no fix of an older fault.
- When the merge is already up to date, change nothing and say so in the change description.
- The Code-Review perch reviews the merge alone: each merge commit's conflict resolution and every commit after the merge, never the parent's own commits the merge brings in.
  Checking the project's goal is no part of this task.
