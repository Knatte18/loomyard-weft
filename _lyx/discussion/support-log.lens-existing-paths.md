```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the existing code paths that read or overwrite each decision's state. Three decisions break against current code, in ways the record does not mention.

**Findings**

1. **BLOCKING — Decisions 4 and 9 (filtered passes recorded as whole).** `greenRun.recordPass` decides whether a pass covers the whole tier from its targets alone, so a filtered dotnet run can be recorded whole.
   - Evidence: in `internal/gatecli/green.go`, `recordPass` writes `Whole = true` in two cases. The first is a command without `{targets}`. The second is an invocation whose targets equal the tier's `all_targets` and whose directory is the tier's own.
   - Under Decision 4 the targets stay the same while `{filter}` narrows the run. A hub whose `all_targets` is its single test project (likely for a small hub) records a class-filtered card or batch pass as whole-green.
   - A tier like `dotnet test X.sln {filter}`, which has `{filter}` but no `{targets}`, hits the first case.
   - Either way, a later `webster`, `merge-in` or `finalize` gate on the same tree carries the full tier without running it. That breaks Decision 9's claim that the whole gates catch the closure's misses.
   - The record says nothing about a narrowed `{filter}` pass and the green write. It also leaves open how a `{filter}` tier without `{targets}` runs under a narrowed dotnet scope. Today's rule runs such a command once, whole; Decision 4 says it runs once per project.

2. **MEDIUM — Decisions 4 and 8 (fingerprint misses the new keys).** `tierFingerprint` hashes a fixed list of fields, so `filter`, `shared_build` and `carry` change nothing unless they are added to it.
   - Evidence: `green.go` `tierFingerprint` builds a struct of exactly About, Command, Dir, Env, AllTargets, Counter, Covers, PrebuiltLyx and Manual.
   - Decision 6 names only the `build` declaration as joining the fingerprint.
   - Effect: changing a tier's `filter` from `Category!=Slow` to something wider carries the old pass under the new filter. Turning `shared_build` on or off also carries.
   - Fix: state that every new tier key that changes what runs joins the fingerprint.

3. **MEDIUM — Decision 6 (`build` key must be in the template).** If the template has no `build` unit, the automatic reconcile after a binary update deletes the hub's `build` declaration.
   - Evidence: the `internal/configengine` doc says a hub file holding units the template lacks loads with a Warn, and `lyx config reconcile` removes them. `PATTERN-config-layers` makes `hubreconcile.Ensure` run reconcile after every binary change.
   - The record seeds `inert_paths: []` in the template but calls `build` "optional" and never says the template holds a `build` unit.
   - After an update, the reconcile prunes `build`. Every tier with `shared_build: true` then fails to load under Decision 6's own refusal.
   - Fix: seed an empty `build` unit in the template, and state that an empty one means "no build".

4. **LOW — Decision 6 (`lyx gate report` counts).** `lyx gate report` totals every run record per gate key as "tier runs". Records with scope form `build` would be counted as tier runs and their wall time added to the row's total. The record says only that the report "shows its time".

**Paths traced**

- **Decision 1 (`dotnet` scoper):**
  - `gateslot/config.go`'s scoper check (`go` or `none` today) needs the new value; no conflict.
  - `gatecli/run.go` opens the green record with `scoperGo = Scoper == "go"`. Under `dotnet`, package entries are neither carried nor written. Survives, and matches "only whole carries".
  - `run.go`'s guard step and `tiers.go`'s `--guards` refusal and card vet step are all gated on `go`. Survives.
- **Decision 2 (card scope):** webster's card gate runs `FindGuardTests` over its own walk; a C# tree has no Go files, so it finds nothing. Survives.
- **Decision 4 (`{filter}`):**
  - `renderTierCommand` fills only `{targets}`, `{args}` and `{parallel}`, so it needs the new placeholder. Explicit targets and darn's tree-scope targets go through it verbatim, with `{filter}` rendering the tier filter alone. Survives.
  - `recordPass` breaks (finding 1) and `tierFingerprint` breaks (finding 2).
- **Decision 5 (`inert_paths`):**
  - `diffBaseGroups` and `newestWholeAncestor` already work with `scoperGo` false. Survives.
  - The carry writes a whole entry that later serves as a diff base, and that tree did effectively pass. Survives.
  - The verifytree command-and-tree skip sits above all this and is untouched. Survives.
- **Decision 6 (build step):**
  - The template must hold `build` (finding 3).
  - A pause hold that kills the verify command ends the build with the call, and the rerun builds again. Survives.
  - A nested gate run with an inherited slot builds again. Survives, at some cost.
  - `stillGreen` and `DirtyPaths` treat `bin/` and `obj/` as dirt only when the repo does not ignore them. That is an existing condition, not a new one.
- **Decision 7 (warm processes):** `gatecli/child.go` kills the process group only when the context is cancelled, not after a normal end. `runproc.End` reaches only processes still below a record. Idle nodes survive, so the record's claim holds.
- **Decision 8 (`carry: false`):** `subtract` and `recordPass` both need the gate. The fingerprint needs it too, or a pass recorded before `carry: false` was set could still be carried (finding 2).
- **Prompts:** `TiersValue` renders tier lines into the loomcli and webstercli prompts, and the new keys are not rendered. Harmless.
- **Shell guard:** `shellguard`'s `refuse_raw` reading of `gate.yaml` is unaffected.

**Tool calls**

1. grep for readers of `Scoper`, `tree_scope` and `TiersValue`: I needed to know who else reads the scoper and tier config.
2. grep in `tiers.go`, `run.go` and `config.go` for `Scoper`, plus `renderTierCommand` and the green function list: I had not seen these bodies.
3. `green.go` lines 60–95: the body of `tierFingerprint`.
4. `green.go` lines 275–390: the bodies of `subtract` and `recordPass`, to see how a pass is judged whole.
5. grep of `gatecli/child.go` and the `configengine` doc: whether a tier's group is killed after a normal end, and what happens to a key the template lacks.
