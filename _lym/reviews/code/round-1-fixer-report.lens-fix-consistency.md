```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix change set agrees with its surroundings.

**Findings**

1. **LOW:** F3's rule in `internal/gatecli/scope.go`, `ownPackageTiers` (a tier that runs whole is not narrowed), was not carried to the sibling statements of the narrowing, which still say every such tier runs the cards' packages alone.
   - The `computeScope` doc in `scope.go` says "each listed tier above tier0 that does not cover tier0 runs the cards' packages alone".
   - `internal/gatecli/doc.go` says "Each other listed tier runs the cards' packages alone, with the main packages when asked, and the call names it on stderr". For a `carry: false` tier, a `--no-carry` call or a command with no target placeholder, the tier now runs whole and the call prints no narrowing line.
   - The same unqualified sentence is in `contracts/specs/loom-plan-spec.md` (the per-batch bullet), in the `gate.yaml` comment in `internal/gateslot/template.yaml`, in the new `--card` help in `internal/gatecli/cli.go` (F6), and in `_lym/landing/summary.md` under "What changes".
   - `doc.go` says elsewhere that a `carry: false` tier runs its command even on an empty scope. A reader can reconcile that case, but the stderr-line claim in the narrowing paragraph is the one most clearly out of step.

2. **LOW:** `_lym/landing/summary.md`, the bullet "`gateslot.BatchGateKey` names the `batch` key for the config check and the scope", is stale after F1.
   - The constant is now also what `goReadGates`, `runplan.RenderBatchGate`, Weaver's batch gate calls and entry name, and `webstercli`'s `loadGateConfig` use.

No other twin was left behind. `git grep gateKeyBatch` finds no remaining reference. The spec's rewritten verify-model sentence agrees with its per-batch bullet.

Tool calls:
- `git grep` for `gateKeyBatch` and for batch-plus-dependents wording, with a `:!_lym` exclusion that git rejected, so it returned nothing. My context had no repo-wide search of docs for the removed identifier.
- The same `git grep` without the exclusion, for the same reason.
