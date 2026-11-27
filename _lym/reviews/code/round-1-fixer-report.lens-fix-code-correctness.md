```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed (lens fix-code-correctness).

1. **MEDIUM**: `internal/gatecli/cli.go`, the Long text "--guards, and a narrower scope while … is go, also run the //lym:guard tests the tiers leave out."
   - The new wording claims something the code does not do for the computed and `--base` scopes.
   - There the guards come from `targets.Guards`, merged with `mergeGuards` in `computeScope`, and `DeriveTargets`/`targetsFor` drop every guard of an impacted package whatever its tier. `scanGuards` and `listedTiers` are never reached on that path.
   - Concrete case: a computed tier0 call over a diff in `cmd/lym` leaves `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (tier1) out of the tier run, and the guard step does not run it either.
   - The old "outside the scope" was accurate. Wording that keeps it and adds the `--card` case would hold; the 2000-byte cap on Long limits the length.
   - The underlying gap in the computed and `--base` scopes predates this fix.

2. **LOW**: `internal/gatecli/scopes_integration_test.go`, the row "--card runs a dependent package's guard in the guard step unless a listed tier is or covers its tier".
   - The "covers" half is never exercised. In the `--tier 1` call, the only guard inside the scope is `TestConsumerGuard`, which is tier1, the listed tier itself. `guardpkg`'s tier0 guard is outside the scope.
   - If `coveredTiers` dropped the `tier.Covers...` append, both assertions would still pass, and so would the rest of the suite as far as I have seen it.
   - A tier0 `//lym:guard` in the `consumer` package, asserted absent from the guard step under `--tier 1`, would pin it.

I traced the rest by hand and found no wrong result or broken caller:
- `scanGuards` filtering per test, including watched-test removal and a scope with no modules (`TestScanGuards`).
- The reordered `--main-packages` listing: `withMainPackages` returns early on nil `mains`.
- The `cardDependents` signature and its one production caller.
- `dependentsFor` with `./lib/`, `.` and a package not yet created.
- The new F1, F2 and F5 assertions each fail with their original defect in place.

Outside the fix, in one sentence: a guard's tier is the lowest `tiergate.Require` in its file, so a tier1 guard in a mixed-tier file is skipped by a tier0 run both inside and outside the scope, as it was before.

Further tool calls: none.
