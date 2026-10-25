```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every way forward the round-3 fix added can be carried out and makes progress.

**Findings**

1. **MEDIUM.** Changed text: D10's new origin-reconcile paragraph ("reports the branch stopped with the pull's way forward") and the new Acceptance criterion ("a hub-scope origin reconcile there, such as `lyx orch start`'s, ... reports the way forward").
   Summary: when `lyx orch start` reports the prime's stop, the `lyx fabric anchor <prime> --set <anchor>` it names cannot run yet, and the fix does not say what must come first.
   Evidence:
   - `internal/orchcli/start.go:253` runs `reconcileHub`, then `:271` runs `dobbyengine.EnsureLive`, and then the orch session launches. The report reaches the operator through `reconcileNote` (`:327`) in the fresh orch's prompt.
   - By then the hub has a live reed session (the orch) and the dobbies just started, with their batten runs.
   - D7's prime-only refusal covers "any live reed session or stored strand in the hub, any live batten run".
   - So the named `--set` is refused from the state the report is read in. It only works after `lyx orch stop`/`lyx reed down` and after the runs finish.
   - The stop should name that order, for example: stop the orch and wait out the hub's runs, then `--set`, then `lyx orch start`. Alternatively, when the prime's weft is stopped, `lyx orch start` could skip `EnsureLive` and the launch, naming the `--set`.

2. **LOW.** Changed text: D10's checkout refusal ("updating the warp checkout by hand to a commit that tracks it, such as by merging the target warp branch into the current one").
   Summary: the step can be done and makes progress, but the example merges the target branch's whole code into the branch being left, and the refusal does not name that cost.
   Evidence:
   - The sentence before it does name a cost, the `--set` committing a new marker "on the branch being left, shared by every machine". The new merge step is a code change on the same branch with no cost named.
   - The merge can also conflict. The refusal gives no way forward for that case, and no lighter route, such as committing the anchor directory's tracked content on the current branch.

Tool calls:
- One `Bash` call: read `internal/orchcli/start.go`'s order of `reconcileHub`, `EnsureLive` and the launch, and `movePrime` in `internal/fabricengine/originreconcilemoves.go`. My context held neither the orch start order nor whether the prime's code branch moves when its records branch stops.
