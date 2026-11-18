```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens review of the Hardener decision record. I traced each decision against the code paths that already read or write the same state.

**Findings**

1. **BLOCKING — D5 (sandbox root under `internal/lymtemp`):** the sandbox hubs get deleted in the middle of a round.
   - The lymtemp doc says each process owns one locked run directory, and `Root.Sweep` deletes the run directories of dead owners.
   - `lym harden sandbox new` is a short-lived CLI process. When it exits, its lock goes with it.
   - The next lym process to start sweeps that directory. That could be the agent's next `lym` call, a gate run, or `tmuxkit.Main` in a tier-2 test.
   - The record never says which long-lived process owns the sandbox root, or where a stable path for it lives.

2. **MEDIUM — D1 with `verify_source: tree_scope`, plus D10:** a hardener run would read darn's config for its verify scope and attempts.
   - `loomCLI.verifySource` in `internal/loomcli/arm.go` returns `darnVerifySource` for any tree_scope recipe, so Finalize's verify scope comes from `darn.yaml`'s `tree_scope` key.
   - `loomrecipe/darnrecipe.go` stamps `DarnVerifyAttempts` onto the verify gate of the build row. Under D10 that is HardenFix, so its attempts also come from `darn.yaml`.
   - The record is silent on both. D3 claims HardenFix's gates are fixed, which this stamping contradicts.

3. **MEDIUM — D9 and D3 (round cap and model rotation from `hardener.yaml`):** the loom wiring does not know the `Harden-Review` segment.
   - `loomrecipe.New` refuses any review segment that has no `Env.Segments` entry.
   - The segment-to-perch table in `internal/loomcli/wiring.go` maps only Discussion-, Plan- and Code-Review to a `loomengine.Perch*`.
   - `reviewBudget` loads only `loom.yaml` and `tapestry.yaml`.
   - The Burler and Bouncer rows refuse model keys; their models come from the loom wiring, which reads `loom.yaml`.
   - So the hardener recipe either fails to build, or its rounds take `loom.yaml`'s budget and models instead of D9's. The record needs a decision that adds a Hardener perch to that wiring, reading `hardener.yaml`.

4. **MEDIUM — D6 (capped or circling campaign escalates through "the Bouncer's existing parent notice and decision verbs"):** the parent's answer cannot reach Harden-Bouncer.
   - `loomrecipe.BouncerRunSubdir` resolves a Bouncer row's run directory by scanning only the tapestry, darn and arras recipes.
   - Harden-Bouncer is in none of them, so the verbs that use this lookup (such as `lym loom circling accept` and `continue`) would not find its run directory.

5. **MEDIUM — D1 ("started by its parent's dobby like any task"):** batten would refuse the task.
   - `battencli/wire.go` (around line 607) has a closed list of child recipes that have a bootstrap verb Run-Shed can start. Anything else gets `ErrUnsupportedChildRecipe`.
   - Other closed recipe lists would also misread a hardener run unless extended:
     - `loomrecipe/names.go`: the embedded-recipe map and the names list
     - `loomrecipe/finalizerow.go`
     - `loomrecipe/recipekeys.go`
     - `reportengine/runs.go`: `runRecipes`
     - `reportengine/timingspans.go`
   - D1 names only the shedcli table and the shedrun vocabulary. As written, dobby cannot start the campaign. The fix is to state the method: every closed recipe allowlist admits `hardener`.

6. **LOW — D2 vs D9 (`hardener.packages` and `hardener.llm_driving`):** these keys are missing from D9's unit.
   - The boardengine doc says `run_settings` maps a module name to that module's config keys, nested as in its YAML file.
   - So `hardener.packages` and `hardener.llm_driving` are keys of `hardener.yaml`, but D9's key list leaves them out.
   - Seeding `run_settings` into a unit that does not declare them is likely to be refused or ignored.

**Paths traced**

- **D1:** shedcli table and shedrun vocabulary survive. The batten child-recipe allowlist breaks. The loomrecipe and report recipe lists break, since they don't list hardener. The arm's verifySource misreads (darn.yaml).
- **D2:** reading `run_settings` works only if the keys are declared in D9's unit; not confirmed.
- **D3 / D9:** the loomrecipe segment budget refuses the build. The wiring's segment-to-perch table and model keys break.
- **D4:** gateslot and `tmuxkit.Main`'s stray check survive. The stamping of verify attempts onto the build row breaks D3's fixed gates.
- **D5:** lymtemp's sweep of dead owners breaks the sandbox root. Reed `-L` socket naming survives.
- **D6:** the Bouncer facts and ledger survive as generic machinery. The parent-decision verbs break through `BouncerRunSubdir`.
- **D7:** not traced (no call left). The `commit_seam` key accepts a new value only if the Bouncer row's seam switch gains it.
- **D10:** `LoomBuildEngines` is read by `shedbuild/loomcontract.go` (survives with the new rule) and `loomrecipe/darnrecipe.go` (misreads, see finding 2).

**Tool calls**

1. `loomcli/arm.go` verifySource, and a grep for recipe-name branches — to see what reads a recipe's name or verify source; my context had neither.
2. `loomrecipe/doc.go` and the segment-budget code in `loomrecipe.go` — to see how a new review segment gets its budget and models.
3. `lymtemp/doc.go` and the `run_settings` docs in boardengine — the sandbox root's lifetime and how settings are keyed were not in context.
4. A grep for `LoomBuildEngines` readers and the segment-to-perch table — to find every reader of the build-row set D10 changes.
5. `loomrecipe/bouncerrows.go` and the wiring's segment table — to check whether parent-decision verbs resolve a new Bouncer row.
