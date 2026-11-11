```yaml
lens: existing-paths
outcome: reported
```

The existing-paths trace found one MEDIUM finding (a new false gate failure from the existing empty-tier check). Every other path I traced survives.

**Paths traced, per decision:**
- **D1 `scope: coverage`**
  - `gateslot` `decodeGates` refuses an unknown scope. A Constraint already names this (older binaries refuse `coverage`), and an acceptance criterion covers it. **Survives.**
  - `withGateScope` (scope.go:107): in its switch, `coverage` falls to default and keeps `--card` ids. D1's "derived exactly as under `diff`" makes it drop them as `diff` does. **Survives.**
  - `TiersValue` / `goReadGates` (tiers.go:135) prints the scope string into prompts as is. **Survives.**
  - The retired-list migration (config.go:547) still writes `scopeDiff`, which keeps the old meaning. **Survives.**
  - Darn `tree_scope`: the hub's darn.yaml holds only `docs` and `tools`, so `internal/` diffs reach the key's own call. D1 leaves explicit targets un-narrowed. **Survives.**
  - `FailedTestsCall` rerun (tiers.go:416) uses explicit targets, so it is never narrowed. **Survives.**
  - The hub's gate.yaml lists `webster-fix`, `darn-write` and `code-burler-fix` explicitly with `diff`, so a template default change does not reach them (Scope Out). **Survives.**
- **D2 (map shape, staleness, marker)**
  - `testmap` `stale` (testmap.go:450) gets the new rules. **Survives.**
  - `dirtyGoFile` (testmap.go:415) checks Go files of the tested directory only; D2 widens the check to every covered package. **Survives.**
  - `changedFiles` (testmappackage.go:204) is widened by D2. **Survives.**
  - `writeFailed` and `writeMap`: D2 says a failed-build map carries the marker. **Survives.**
  - `WriteMap` through `internal/state` keeps keys it does not know. Rewrites by an older binary are caught by the marker's `Commit`/`BuiltAt` check. **Survives.**
  - Redundancy `mappedPackages` / `checkMapBlocks` (redundancy.go:280,335) and the redundancy judgment keep own-package keys only. **Survives.**
  - `mapDrift` (redundancy.go:349) stays on own-package `Files`. **Survives.**
  - The prime dobby's refresh trigger and the exit record's `unbuilt` count: the new staleness inflates the backlog. Open risks and the parent question name this. **Survives.**
- **D6 (green record and invocations)**
  - Green `covered` and `argsMatch` (green.go:182,224): an empty-args package entry covers a later `-run ^(...)$` request, because `isGuardRunFilter` matches that shape. This is correct: a whole run includes the selected tests. **Survives.**
  - `newestWholeAncestor` (green.go:202) ignores entries that have args. **Survives.**
  - `with()` pruning: a whole entry with args is never kept as a base. **Survives.**
  - `subtract` in a later scope-`all` `finalize` call on the same tree: the selected entry has args and the finalize request has none, so `argsMatch` needs an exact match and fails. Selected packages are not carried. **Survives**, and the D1 bound holds.
  - `recordPass` and `coversAllTargets`: the selected packages do not count as covering `all_targets`. **Survives.**
  - The tier run's empty-tier check, `emptyTierFailure` (tierrun.go:387), does not survive; see Findings.
- **D8**
  - The maps directory is shared per machine (coveragemap.go:19). An adopted run on another machine finds no maps, and D5 widens. **Survives.**

**Findings**

1. **MEDIUM**, class `design`, at D3 (the union) together with D6 (the selected invocation).
   - **Summary:** a narrowed package can run zero tests in the selected invocation. The existing empty-tier check then fails the gate with a false "the tier's command lacks its tag" refusal. The record says nothing about this check.
   - **Mechanism:** `emptyTierFailure` exempts a run only when the call's own words after `--` hold a filter. It does not read the run's args: "The run's args are not read, as the guard step appends its own `-run` filter there" (tierrun.go:383–388). It fails when a package ran 0 tests while `holdsTierTest` finds tier tests in it, and a test named by the watch skip does not count toward those (tierrun.go:385, 403–416).
   - **Case 1 (watch skip):** the D3 union does not drop tests the watch skip names, and D1 keeps watch skips "as under diff". Take an importing package whose only union member is a `//lym:guard` + `//lym:watch` test that this diff leaves alone. It enters the selected invocation with `-run ^(G)$` plus the `-skip ^(G)$` watch filter and runs 0 tests. It still holds other tier tests, so the gate fails. Under `diff` the same package runs whole minus G, so it passes.
   - **Case 2 (guard of another tier):** D3's "every `//lym:guard` test of the package" is not limited to the tier. Today the guard step groups guards by their file's tier (scope.go:453, `guardsByTier`). A tier0 selected invocation naming only a tier1-file guard compiles no such test under tier0's tag and also runs 0 tests.
   - **Fix:** state in D3 that the union holds only tests of the tier and drops the tests the watch skip names, so a package whose union empties counts as "none selected". Alternatively, state in D6 that the selected invocation is exempt from the empty-tier check, as a filtered call is.

Tool calls (all six were Bash greps or reads):
1. Grepped production code for readers of a gate key's scope value and of `"diff"`, to find scope readers beyond `scope.go` and `config.go`, which my context did not show.
2. Grepped for `//go:embed` paths, to see whether any package embeds files from a subdirectory that D2's per-directory file record would miss.
3. Grepped `counters.go` and `tierrun.go` for how a tier that ran no test is treated, because the selected invocation can run zero tests and no file I had read covered that.
4. Read `tierrun.go` lines 370–419 (`emptyTierFailure`), the check that grep pointed to, to see when it exempts a filtered run.
5. Grepped stencils, specs and Go for readers of `TiersValue` and for prose stating the scope values, the prompt-side readers of the rendered scope.
6. Read the hub's `darn.yaml` `tree_scope` and its reader in `darnengine`, to learn whether Darn-Write usually passes explicit targets, which would bypass narrowing.
