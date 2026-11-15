# Worktree .scratch is a symlink into another worktree's .lym/scratch, breaking relative project references

`scratch-symlink-other-worktree` · Task · bug, fabric · running · Darn-Write

Imported from [issue #120](https://github.com/Knatte18/loomyard/issues/120).

In a task worktree, `.scratch/` in the working directory is a symlink into another worktree's `.lym/scratch`, not a directory inside the worktree.

Why it is abnormal: the agent instructions say scratch files go under `.scratch/` in the working directory. A card asked for a no-git run under `.scratch/`. Building a project there fails, because its relative project references resolve from the symlink target in the other worktree and not from the working directory. The agent worked around it by copying the project into a scratchpad nested in another git repository.
The symlink also puts one task's scratch files in another worktree's tree, which conflicts with the rule to work only in your own worktree.

What to do: make `.scratch/` a real directory inside each task worktree, or document that it is a link and tell cards not to build projects with relative references under it.

Provenance: task `td-string-inputs`, note `07-string-property-profile.md`.
