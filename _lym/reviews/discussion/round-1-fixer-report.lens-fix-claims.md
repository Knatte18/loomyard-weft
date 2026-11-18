```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, that every claim the fix added or rewrote holds against the code at HEAD.

1. **MEDIUM.** Changed text: D5 Bounds, "since fabric's leftover cleaners refuse a directory holding a `.git` at any depth (`internal/fabricengine/destroy.go`), as every sandbox hub does".
   Summary: the two cleaners check `.git` differently, so "at any depth" does not hold for both.
   Evidence: `RemoveStrayTaskPath` checks only the top level, through `pathPresent(filepath.Join(target, ".git"))` (`internal/fabricengine/destroy.go:1411`). Only `ignoredTaskPathDeletable` reports false for a `.git` "at any depth" (`destroy.go:1426`). Both cleaners also act only on a task worktree path under the hub (`WorktreePath(l, slug)`), not on a run worktree's `.lym` subtree, so it is unproven that the sandbox hubs are what blocks teardown. A correct statement would cite `ignoredTaskPathDeletable`'s any-depth refusal for a leftover task worktree path.

2. **LOW.** Changed text: D4, "as `gateslot.AcquireSlots` clamps it".
   Summary: `AcquireSlots` is a method on `*Pool`, not a package-level function, so the identifier as cited does not exist.
   Evidence: `internal/gateslot/pool.go:179`, `func (p *Pool) AcquireSlots(ctx context.Context, h Holder, count int) (*Lease, error)`. The correct citation is `gateslot.Pool.AcquireSlots`.

The other added claims hold at HEAD:
- `pickRound` sits behind `RoundModels.Pick`.
- `childRecipeSupported` is in `internal/battencli/wire.go`.
- `skillLists` and the test rows `UnknownRole` and `PairWithUnknownRole` exist. `efforts` refuses only role/model pair keys naming an unknown role, so stripping the keys that name a retired role is correct.
- `configreg.Reinterpretation`'s `Rewrite` rewrites the hub base file's bytes.
- The module name is `agents` (`agentconfig.ModuleName`).
- The gate template ships `code-burler-fix` with tiers `[0]`, and a key may name only a declared tier once any tier is declared.
- Explicit targets beside `--gate` replace the key's scope (`internal/gatecli/cli.go:74`).
- `lym config gate --set` exists.
- Multi-slot acquires clamp to every slot and never grant part, and a nested run inherits its parent's slot (`internal/gateslot/doc.go`).
- `reviewBudget` feeds `recipeRouting`, which serves routing and status.
- `runproc.End(dir, owner, callerPID)` exists.
- `reedengine.ServerName` hashes the absolute path, and its readable half is `filepath.Base`.
- `darnTreeScope` runs the row's key over the touched trees' targets when the diff lies wholly under the configured trees.

Tool calls:
- Read `internal/fabricengine/destroy.go:1384-1432`, `internal/loomcli/arm.go:290-306` and `internal/configreg/configreg.go:100-135`. My context lacked the cleaners' full `.git` checks, the caller of `reviewBudget` and the `Reinterpretation` `Rewrite` contract.
- Grepped the `AcquireSlots` signature and the `runproc.End` doc. My context lacked `AcquireSlots`' receiver and `End`'s signature context.
