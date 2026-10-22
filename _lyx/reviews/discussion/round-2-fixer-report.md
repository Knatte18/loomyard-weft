# Round 2 fixer report — discussion

## Changes

- **F1** (MEDIUM, consistency) — fixed as proposed, plus one wording addition from the self-check.
  In `decision-record.md`, the Acceptance criteria bullet that set a standalone worktree file under a hub file is replaced by two bullets.
  The first covers the hub file through `lyx config reconcile --layer default --apply` from the hub's prime worktree.
  The second covers a standalone worktree file holding `lens_set: csharp` through `lyx config reconcile --apply`.
  The premise holds: `ReconcileAll` sends a hub worktree file to `reconcileSparseFile` (`internal/configsync/configsync.go:401`), and only `reconcileBaseWorktreeFile` and `ReconcileHubWideAt` (`DropTemplateEqualNames`, `configsync.go:917`) drop template-equal names.
- **F2** (LOW, consistency) — fixed differently.
  D3 gains a statement that `configengine.Set` refuses, before anything is written, a write that leaves the selector key in a hub worktree file.
  `configengine.Edit` refuses such a save after the editor writes it and puts the file back, as it does for a refused local save.
  Both give D3's refusal, which names the removal step only when the file held the key before the write.
  So a hub-worktree `--set lens_set=<name>` gives that refusal alone, without `newLoadFailure`'s generic "way forward: --set the key to a value it takes" (`internal/configcli/configcli.go:295`).
  A new acceptance criterion pins this: in a hub worktree whose `lenses.yaml` holds no `lens_set`, `lyx config lenses --set lens_set=csharp` writes nothing and is refused naming `--layer default` from the prime, with no other way forward.
  Three parts of the proposed text were not taken:
  - "Any layer above the base" would cover the local layer, where D3 already says `ValidateLocal`'s Guard refusal runs first, so "the same refusal" would contradict D3. The rule is scoped to a hub worktree file.
  - "With the same refusal and way forward" would tell an operator whose file never held the key to remove it from that file. The removal step is named only when the file held the key.
  - "The worktree editor's save … before anything is written" cannot hold, because `configengine.Edit` at the worktree layer runs `EditPath` on the real file (`internal/configengine/edit.go:133`), so a save is refused after the write and rolled back.

## Self-check

- fix-consistency — no findings.
- fix-proposal, LOW: F1's hub-file reconcile criterion omitted "from the hub's prime worktree", and `--apply` is guarded to the prime (`internal/configcli/hubwide.go:58`, `:195`). Fixed as proposed.
- fix-claims, MEDIUM: D3's new sentence claimed `configengine.Edit` refuses before anything is written, but `Edit` writes through `EditPath` first (`internal/configengine/edit.go:133`). Fixed as proposed: `Edit` refuses after the save and restores the previous bytes, or removes a file it created.
- fix-way-forward — no findings.

## Disputed

None.

## Deferred

None.
