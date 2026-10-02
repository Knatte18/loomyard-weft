# A run's parent is the worktree it is seeded from; batten survives pauses; parent-review loops

`run-parentage` · Planned · bug · running · Webster

## Goal

A run knows its parent from where it was seeded, and batten carries a run through pauses instead of dying, so a run needs no special shell, environment variable or restart to reach the orch session.

## Bugs

- #341: a run's `parent` is the caller's `LYX_STRAND_NAME`, so a run started from a plain shell has no parent and its escalations reach no one.
- #338: a paused or blocked inner run makes `lyx batten run` exit `failed`, leaving no watcher to land or tear down the pair.

## Fix direction

- #341: creating and seeding a pair records the worktree it is seeded from as `parent`.
  A driver that messages upward resolves the orch session running in that worktree.
  This rewrites the Agent Name Invariant in `CONSTRAINTS.md` in the same commit.
  Open for Discussion: what a message does when the parent worktree has no orch session (queue until one appears, or log only).
- #338: batten waits through `paused` and `blocked`, tells the parent once, and carries on when the run is `running` again.
- Parent-review gets more than one round: today `attempts: 1` with `pass_on_cap: true` lets the rewrite after a reject through unseen.
  The parent reviews each rewrite until it approves, up to a cap Discussion sets, and escalates at the cap instead of passing.

## Done when

- A run started with plain `lyx batten run <slug>` from the prime has the prime as parent, and its parent-review request reaches the orch session.
- Pausing and resuming an inner run leaves its batten watcher alive.
- A rejected discussion comes back to the parent for review until approved or the cap escalates.
- CLAUDE.md's TEMPORARY items for #341 and #338 are deleted (the issues are already closed into this task).
