```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness of the diff, one case per changed function or branch.

**List**
- `fixRoute` (`verify.go`), sampler rows: `Rows` holds Sampler-Session, so the route is `goto --to Sampler-Session`. Correct. The loop overwrites `row` in order, so a run holding both writer rows would route to Sampler-Session. No recipe holds both, so this does not matter.
- `fixRoute`, tapestry rows: no writer row, so `Webster`. Correct. Darn rows: `Darn-Write`. Correct. Empty `Rows`: `Webster`. Correct.
- `wire` (`loomcli/wiring.go`), `hasPlan`: the old condition `!darnRun` is replaced by `hasPlan`. This is correct for tapestry (has PlanWrite, plan dir set), darn (none, as before) and sampler (none). Only the darn, tapestry and sampler recipes contain Drift rows, so only they reach the drift plan dir. No other recipe is affected.
- `samplerStamp` / `EntryStamp`, drift rows skipped: the predecessor is derived from `OnDone`, so it is now Drift-Expect. A Skipped Drift-Expect entry counts as a predecessor (`isPredecessorEntry` accepts Skipped), and the stamp is 1. Correct, and the updated `wiring_darn_test.go` case pins it.
- `EntryStamp`, goto from a Drift-Verify Stuck: the Goto entry for Sampler-Session arrives while `atRow` is false (the Drift-Verify entry hit the default case), so the stamp is raised. The re-entered session therefore gets a new stamp. Correct.
- Sampler recipe entry gates, scope `""`, `off` or `full` with `drift.enabled` true or false: only `enabled && scope == full` enters the rows. Correct, covered by `TestDriftRows_EntryGate`.
- Sampler recipe chain `on_done` (Record-Write → Drift-Baseline → Drift-Await → Drift-Expect → Sampler-Session → Drift-Verify → Code-Bouncer → … → Drift-Recheck → Describe): the order is consistent. When `sampler.code_review` is false, Code-Bouncer routes on to Drift-Recheck, which re-checks code the perch never changed. This is harmless; it matches darn's Drift-Recheck behaviour.
- `driftcli` refusal strings: only the text changed, so behaviour is unchanged.

**Findings**
- None.

**Tool calls**
1. Listed `contracts/recipes`, grepped for entry-gate parsing and read `samplerStamp`. My context lacked the stamp wiring and the list of recipes.
2. Grepped which recipes contain Drift rows and read `EntryStamp`. My context lacked the stamp logic and the extent of the `hasPlan` change.
