```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state or setting the record changes, I traced the code that already writes or reads it and checked whether the decision survives.

**Paths traced, per decision**

- D3, an in-flight run whose `scope.json` holds `partial` with a `Skip` list (`internal/driftengine/records.go:21,73`). `settleConfiguredScope` keeps a scope that is already recorded (`baseline.go:139-145`), so the record keeps saying `partial`. D3 covers only a recorded `off` or `full`. **Breaks.**
- D3, a run's config holding `scope: partial`. Today `checkScope` reads it (`config.go:190-200`), and D3 says it becomes `full` with a Warn. Holds, provided the new `checkScope` admits it.
- D3, Drift-Baseline re-entered after the scope is recorded. It keeps the record (`baseline.go:139`). Survives.
- D15, the retired `skip` and `verify_max_runs` in `run_settings`. Both are `CategoryNone` (`driftengine/config.go:121,123`), and configengine's retired-key rule lets a `CategoryNone` retired key load and drops it. **Breaks** the "refused at seeding" claim.
- D15, `diff` and `key_patterns` refused in `run_settings`. This works through a category other than `CategoryNone`, as `ValidateLocal` refuses (`configengine/doc.go:70-73`). Survives.
- D14/D15, the hub `gate.yaml` key `drift-verify`. It is read through the Drift-Verify row's `gate_key` (`tapestry-recipe.yaml:342`, `darn-recipe.yaml:127`), `goReadGates` (`gateslot/tiers.go:113`), the gateslot template (`template.yaml:30`) and `envkit.go:284`. D14 removes the verify gate entry, and nothing in the record retires or redefines the key. **Breaks (orphaned).**
- D13, `driftengine.Verdict` called from landing (`loomcli/landingdeps.go:244`, `loomcli/wiring.go:809`) for a run past Drift-Recheck when this lands. Its `scope.json` holds the old `Compared*`, `Accepted` and `Waived*` fields and no check result. D13 renders only the new fields. **Breaks** for in-flight landings.
- D6, "a run past Drift-Expect". The drift deps already carry a `Status` seam (`driftwiring.go:60-66`), which holds `CurrentProducer` and `History` (`shedengine/status.go:136-161`). A run from before this lands never has Drift-Expect in its `History`. The record does not say which of the two fields decides. The rule can be built, but D6 leaves it unstated (see finding 5).
- D2/D7, the compare and capture requests. Today they pass `scope.Skip` through `driftwiring.go:150-153,185-190`. D14 removes `Skip`, so for a new run nothing reads it. Survives, except for the in-flight partial case under D3.
- D7, the capture lock against a detached baseline capture. A main capture runs only at Drift-Baseline or a Drift-Await halt (`checkmain.go:23-31`), and a check runs only after Drift-Await. Survives.
- D8, a Master already spawned by the old binary. Its `GateSpec` was fixed at spawn (`runlevel.go:790`), so it runs without the `drift` entry until it is respawned. Survives: Drift-Verify still checks.

**Findings**

1. **MEDIUM**, design, D3 and Open risks. An in-flight run with a recorded scope of `partial` is not covered.
   `settleConfiguredScope` keeps the recorded `partial` (`internal/driftengine/baseline.go:139-145`). Its baseline capture skipped the `Skip` tests (`loomcli/driftwiring.go:150-153`, `capture.go:39-40`), so those cases are missing from the baseline directory. Under D1 each one diffs as an `added case`. No expectation covers it, so Drift-Verify goes Stuck.
   The record says nothing on how the new code reads a recorded `partial` and its `Skip`: as `full`, by dropping the skipped cases, or by refusing with a way forward. D3 only says a recorded `off` or `full` is kept.

2. **MEDIUM**, design, D15. The claim that a retired key is refused at seeding misreads the retired-key mechanism.
   `skip` and `verify_max_runs` are `CategoryNone` (`internal/driftengine/config.go:121,123`). configengine judges a retired key by its old category, so a `CategoryNone` one is let through and dropped. Only a refused unit gets the `RetiredWayForward` text (`internal/configengine/doc.go:75-78`).
   As written, a `run_settings` entry that carries `drift.skip` loads silently, with no refusal. D15 needs to state the category or check that produces the refusal.

3. **LOW**, design, D14/D15. The hub `gate.yaml` key `drift-verify` is left orphaned.
   It is read today through Drift-Verify's `gate_key` (`contracts/recipes/tapestry-recipe.yaml:342`, `contracts/recipes/darn-recipe.yaml:127`) and is listed in `goReadGates` (`internal/gateslot/tiers.go:113`) and the template (`internal/gateslot/template.yaml:30`). D14 removes the verify gate entry, and D15 covers only `drift.yaml`. The record does not say whether the key is retired, kept for D7's slot, or reused.

4. **LOW**, design, D13 and Open risks. A run already past Drift-Recheck when this lands loses its record of the old compare and acceptances.
   Such a run lands through `driftengine.Verdict` (`internal/loomcli/landingdeps.go:244`). Its `scope.json` carries only the old compare, acceptance and waiver fields (`verdict.go:41-89`). D13 renders only the new check fields, so the pull request drops the old compare result and the parent's rulings. The Open risks line covers only a run past Drift-Expect that still has to reach Drift-Verify.

5. **LOW**, design, D6. The record does not say how code tells that a run is past Drift-Expect.
   The `Status` seam exists (`internal/loomcli/driftwiring.go:60-66`, `internal/shedengine/status.go:136-161`). But a run from before this lands has no Drift-Expect entry in its `History`, so the rule has to be the position of `CurrentProducer` in the recipe's row order. The record names neither the source nor the rule.

**Tool calls**

1. Searched for `drift-verify` across `internal` and `contracts`, because the readers of the gate key were not in context.
2. Searched for scope-record and verdict readers outside driftengine, the runprofile drift block, and the configengine retired-key code, because the outside readers and the retired-key semantics were not in context.
3. Read the drift config `Categories` (`driftengine/config.go:112-132`), configengine `doc.go:70-82` and the category definitions, because the retired-key category rule was not in context.
4. Read `loomcli/driftwiring.go:140-205`, because the compare and capture wiring that reads `scope.Skip` was not in context.
