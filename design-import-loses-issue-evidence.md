# Board entry imported from an issue loses the evidence the issue names, and an issue body can arrive empty

`import-loses-issue-evidence` · Task · enhancement, board, selfreport · running · Drift-Verify

Imported from [issue #148](https://github.com/Knatte18/loomyard/issues/148).

## What happened

An agent was given a task imported from a GitHub issue.
The task said to check a shuttle idle reading against `startup-capture.txt` in two run directories under `.lym/shuttle/`.
Neither directory existed on the machine: the worktree of the source run was gone, and no hub log held the failure.
The agent spent a long time searching for the evidence before concluding it was unrecoverable.
A second part of the task was imported from an issue whose body was empty, so only the title was available to work from.

## Why this is abnormal

The board entry points at evidence (captures, log lines) that lives only in a run directory that can be deleted with its worktree.
Once the worktree is gone, the entry names files nobody can read.
An issue body that arrives empty gives the entry nothing to act on, and nothing flags the loss.

## What to do

- When the board imports an entry from an issue, copy the evidence the issue names into the entry, or into a file the entry references that outlives the source worktree.
- Flag an import whose issue body is empty, so the empty entry is not mistaken for a complete one.
- Consider a hint at issue-creation time (`lym selfreport create`) to include captured evidence inline, since run directories are not durable.

Provenance: task slug `shuttle-start-idle-and-prompt`; note file `Darn-Write.md`.