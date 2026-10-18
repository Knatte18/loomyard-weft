# An anchor per umbrella

`umbrella-anchor` · Note · enhancement, fabric, config, work-laptop

## Seen (2026-10-18)

The anchor (`fabric clone --subpath`) is set once per hub and decides where `_lyx`, `.lyx/` and config sit in every worktree of the hub.

## Idea

Let an umbrella (`orch-any-worktree`) have an anchor of its own, a different subdirectory of the repository, inherited by every task under it as it inherits the branch.
Fabric, batten and loom would then read the anchor from the task's umbrella rather than from the hub.

## Operator view, not a decision (2026-10-18)

Hard to change now; a low-priority task for later.
