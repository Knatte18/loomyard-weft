# Board view lists a done beam-project task under P0

`board-done-beam-task-under-p0` · Task · bug, board · running · Code-Burler-Review

Imported from [issue #188](https://github.com/Knatte18/loomyard/issues/188).

Seen: lym board list --text showed a task that had landed in a beam project under P0, the root project, instead of under its beam.

Workaround: none; the entry was left until prune.

Expected: a done task stays listed under its own project until it is pruned.