```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 change set.

1. **BLOCKING** — `reachStep.caveats`, the new `unmappedTierNames(tests, mappedTierConfigs(s.cfg))`. The change made `caveats` depend on `s.cfg`, but the other site that builds a step for it was not updated.
   - `internal/gatecli/reachverb.go` (in `runReach`) builds `step := reachStep{location: location, root: location.WorktreePath()}` with no `cfg`, then calls `step.describe` through `runReachFunction`.
   - A zero config declares no tier, so `lym gate reach <file>#<Func>` on an unreached function now reports every tiered test as unmapped, for example "tests of p in tier0 and tier1, which no map holds, may run it unseen". This is false.
   - `runReach` already holds the loaded `cfg`, so passing it into the literal fixes this.
   - The gate passed, so no reach-verb test asserts this caveat.

2. **LOW** — `_lym/landing/summary.md`, `## Tests`. The `TestUnseenTestCaveats` bullet says "An unresolved test with an outcome counts under its outcome". F4 made an unresolved test with `OutcomeNoCoverage` count as unresolved, so the sentence is now wrong for that outcome. The section also omits the new coverage: `TestUnmappedTierNames`, the `UnseenLinkerTests` row, the index-version row, and the older-format and linker rows of the reach integration test.

3. **LOW** — `internal/gatecli/doc.go`, Mutate verb section. "A package is judged only with a fresh map: one the refresh wrote, or a map whose file hashes match the worktree." In a task worktree the refresh now also treats an older-format map whose hashes match as needing a rebuild. If that rebuild is left unrefreshed (budget or slot), `freshMaps` does not judge the package, because it goes by the refresh's reason (`mutate.go`, `freshMaps` doc). The sentence still says matching hashes are enough.

Tool calls:
- One `Bash` read of `reachverb.go` lines 120-200, plus greps of `reachverb.go`, `testcover/doc.go` and `reachverb_integration_test.go`. My context held only a one-line grep hit for the reach verb's step construction.
