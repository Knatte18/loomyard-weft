# The orch role explains how to make a project into a beam

The orch role listed `lym beam create <slug>` and called a beam "a project entry made a beam", but held no procedure.
In the Models hub this let the prime orch start project P1 as an ordinary task run, read the missing worktree as the fault, patch it with `lym fabric add` and then try to delete the worktree directory by hand.

`contracts/orchrole/orch-role.md` gains a "Making a beam" section after the run loop. It states:

- the two steps, `lym board upsert` with `"kind":"project"` and then `lym beam create <slug>`, run from the prime or the parent beam and never from a task worktree, with `--anchor <dir>` for a beam's own anchor;
- that only `lym beam create` starts a beam's run, since dobby starts no project entry;
- the run's shape: the batten run's Worktree-Create makes the worktree unmarked, Seed-Child seeds the pawl run, and the pawl run's Beam-Up marks the beam and starts its dobby and orch, so the beam's orch appears after Beam-Up; a task run has the same batten rows but no Beam-Up;
- that `lym beam start <slug>` brings an existing beam back up, refuses a task worktree and never makes a beam;
- how to tell a project running as a task run from a beam through `lym batten status <slug>`'s current producer, and the way forward, `lym batten cancel <slug> --remove` on the operator's word and then `lym beam create <slug>`;
- never `lym fabric add` and never a hand deletion of a worktree directory to repair a run: on a refusal, stop and tell the operator.

## Design decisions

- The way forward names `lym batten cancel <slug> --remove`, where the board entry names the cancel without the flag: a cancel that keeps the pair needs `--remove` before the slug runs again, so the bare cancel would leave the operator one refusal short of `lym beam create`.
- The weakness behind issue #9, a batten pass that fails forever when the worktree is gone, is left out as the entry asks: the board entry `batten-remote-order` already covers it, so no new note is filed.
