# The orch role explains how to make a project into a beam

`orch-role-making-a-beam` · Task · enhancement, orch, pawl · running · Drift-Verify

Seen in the Models hub: its prime orch could not make project P1 into a beam, because the orch role it follows never says how.

The role lists `lym beam create <slug>` in its command list and says a beam is "a project entry made a beam", but it holds no procedure, so the agent read a missing worktree as the fault and patched it by hand.

## What went wrong

- P1 was started as an ordinary task run (rows Worktree-Create, Seed-Child, …, Run-Shed) instead of a pawl run (Worktree-Create, Beam-Up), and the agent read only that the worktree was missing.
- It used `lym fabric add`, which only gave the wrong run a worktree back; a beam never comes of it.
- When lym refused, it tried `rm` on the directory itself instead of stopping to ask.
- The way out was `lym batten cancel` on the wrong run, then `lym beam create` from the prime.

## Design

Add a "Making a beam" section to `contracts/orchrole/orch-role.md`:

1. A beam is a board project entry that `lym beam create <slug>` turns into a beam: `lym board upsert` with `"kind":"project"` first, then `lym beam create <slug>`, run only from the prime or a parent beam, never from a task worktree.
2. The verb starts a batten run for the project whose Worktree-Create row makes the worktree, unmarked, and whose Seed-Child row seeds the inner pawl run in it; the pawl run's Beam-Up row marks the worktree a beam and starts the beam's own dobby and orch.
   The orch appears only after Beam-Up, so the operator attaches to a beam later than the create.
   A task run has the same batten rows but an inner tapestry or darn run, which has no Beam-Up, so it never becomes a beam.
3. `--anchor <dir>` gives the beam its own anchor; without it the beam inherits its parent's.
4. `lym beam start <slug>` brings an existing beam back up; it refuses a task pair, and never makes a beam.
5. A project that already runs as a task run is not a beam: `lym batten status <slug>` shows the rows, and the way forward is `lym batten cancel <slug>` with the operator's word, then `lym beam create`.
6. Never `lym fabric add`, and never delete a worktree directory by hand, to repair a run; when lym refuses, stop and tell the operator.

Fold the weakness behind issue #9 (a batten pass that fails forever when the worktree is gone) in only as a separate finding: its premise in the Models case was the wrong run kind, not the missing worktree.
