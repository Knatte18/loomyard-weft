# Beam promote keeps the Discussion record as part of the beam

`beam-promote-keeps-discussion` · Note · enhancement, workspaces, batten, loom

## Seen (2026-10-25)

`lyx beam promote <slug>` ends the task's run for good, keeps the code commits on the pair's branch, and resets the pair's records to a fresh fork of its parent's.
The run's discussion, plan and run records leave the worktree and stay only in the records history.

## Idea (operator, 2026-10-25)

The likely moment to promote is Discussion finding that the task is so large it should be a project.
Then everything the Discussion writer found must be kept: its decision record and what it learned become part of the beam, as the beam's starting design, not something left behind in history.
Note `loom-beam-darns` is a related shape, where a large loom task turns its own worktree into a beam and runs the pieces inside it.

## Until then (operator, 2026-10-25)

Beams are made with `lyx beam create` on a project entry only; `lyx beam promote` is not used.
