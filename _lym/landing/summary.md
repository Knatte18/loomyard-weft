# Orch role: how to make a project into a beam

The orch role in `contracts/orchrole/orch-role.md` listed `lym beam create <slug>` but held no procedure for making a beam.
In the Models hub the prime orch started project P1 as an ordinary task run, read a missing worktree as the fault, patched it with `lym fabric add`, and then tried to `rm` the directory when lym refused.
A beam never came of it.

This change adds a "Making a beam" section to the role.

- A beam is made on the operator's go-ahead in two steps, run from the prime or a parent beam, never from a task worktree: `lym board upsert` with `"kind":"project"`, then `lym beam create <slug>`.
- `--anchor <dir>` gives the beam its own anchor; without it the beam inherits its parent's.
- The section explains the run the verb starts: Worktree-Create makes the worktree unmarked, Seed-Child seeds the pawl run in it, and that run's Beam-Up row marks the worktree a beam and starts its dobby and orch.
  The beam's orch therefore appears only after Beam-Up.
- A task run has the same batten rows but an inner tapestry, arras, darn or sampler run with no Beam-Up, so it never becomes a beam.
- `lym beam start <slug>` brings an existing beam back up; it refuses a task worktree and never makes a beam.
- A project that ran as a task run is not repaired.
  The way forward is `lym batten cancel <slug> --remove` on the operator's word, then `lym beam create <slug>`.
- The role forbids `lym fabric add` and deleting a worktree directory by hand as repairs; when lym refuses a step, the orch stops and tells the operator.

Only the role text changes; no code does.

## Design decisions

- `lym batten status <slug>` reports the batten run's rows, which are the same for a task run and a beam, so the role says it cannot tell them apart.
  The role points to the entry's `kind` (`lym board get <slug>`) and to the inner run's current producer (`lym loom status` in the worktree).
  This departs from the board entry's draft, which sent the orch to `batten status` for the distinction.
- The weakness behind issue #9 (a batten pass that fails forever when the worktree is gone) is not folded into this change.
  In the Models case the cause was the wrong run kind, not the missing worktree, so it stays a separate finding.

## To check by hand

- `status-read-cannot-tell-task-run-from-beam`: confirm that the "Making a beam" section's claim that `lym batten status` cannot separate a task run from a beam, and its use of the entry `kind` and `lym loom status` instead, matches how those commands behave.
- Confirm that `lym batten cancel <slug> --remove` is the right form for clearing a task run before `lym beam create`, and that the list of task inner runs (tapestry, arras, darn, sampler) is complete.
- No command from the new section was run; the change is documentation only.
