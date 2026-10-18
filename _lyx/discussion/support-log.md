# Support log — config-migrate-hub-layer

## Interview

A darn run holds no interview: the writer read the board entry, the issue it imports and the code, and made every call itself.

## Rejected alternatives

- Make the bare `lyx config reconcile` in the prime also reconcile the default layer: a pair's dry run would then report board changes it can never apply, and `--apply` would mix a fabric-synced worktree write with a guarded board commit in one verb.
- Add a reconcile step at landing or to the deploy command: the hub walk at the first start verb after a binary change already runs the same migration over the board dir, and `update-plugins.sh` is production tooling no task changes.
- Refuse a load whose hub file holds unknown keys: every run between a binary change and the reconcile would fail, where the old values only lose effect.
- Warn on every load: a loader in a poll loop would flood stderr and the trace, against `PATTERN-log-levels`.
- Write the removal of the hub's preserved keys as a commit of this task: the hub walk already removed them, and a task never edits the hub's board files directly.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Which entry point reaches the hub layer — a flag on `reconcile`, a new verb, or the landing? Self-pick: `--layer default` on `reconcile`, reusing the flag `lyx config <module>` already takes for the board dir.
- Q2: Should `--apply --layer default` seed absent per-worktree repo defaults? Self-pick: yes, as the walk does, so the two paths leave the board in the same state.
- Q3: Is the operator decision to remove the preserved keys still open? Resolved: the hub's `loom.yaml` and `darn.yaml` hold none of them since board commit `aa9f100c5`.
- Q4: Which layers does the load warning cover? Self-pick: default and worktree; the local layer already refuses unknown keys.
