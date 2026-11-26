```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff changes on concrete inputs.

**Cases run**

| Function / branch | Case run | Result |
|---|---|---|
| `mapPackage` spawn branch | verdict `{OutOfProcess, Unresolved}`: entered unrun | correct |
| `mapPackage` spawn branch | verdict `{Unresolved}`: goes to `toRun` | correct |
| `mapPackage` spawn branch | no verdict: runs as before | correct |
| `mapPackage` spawn branch | every test unresolved: a build now happens, where it used to be a hash-only write | correct |
| `runTest` | `p.spawns` has no entry for the test: zero verdict; the alone-rerun sets the spawn the same way | correct |
| `fillMapTests` | unresolved run that passes and covers nothing: Spawn kept, `OutcomeNoCoverage` set | correct |
| `fillMapTests` | unresolved run that fails: Spawn and outcome kept | correct |
| `MapTest.HoldsCoverage` | unresolved with no outcome: true | correct |
| `MapTest.HoldsCoverage` | out of process: false | correct |
| `MapTest.HoldsCoverage` | v2 map, unresolved entry with no blocks: true, reaches nothing | correct |
| `buildTable` | Unseen collected for unresolved, out-of-process, failed, timed-out and skipped entries; sorted; `NoCoverage` entry with no mark left out | correct |
| `buildTable` | package with no tests: Unseen is nil | correct |
| `mayRunUnseen` | unresolved + `OutcomeNoCoverage`: true | correct, but see finding 2 |
| `unseenCause` | out of process | correct |
| `unseenCause` | failed / skipped / timed out | correct |
| `unseenCause` | unresolved, no outcome | correct |
| `unseenCause` | unresolved + `OutcomeNoCoverage`: names "covered nothing" | wrong cause (finding 2) |
| `unseenTestCaveats` | nil input: nil | correct |
| `unseenTestCaveats` | several causes: sorted by cause string, names sorted within a cause | correct |
| `namedTests` | 5 tests | correct |
| `namedTests` | 6 tests: "… and 1 more" | correct |
| `unmappedTierNames` | nil (scan error): no caveat, error recorded | correct |
| `unmappedTierNames` | tiers {2, 3, 2}: "tier2 and tier3" | correct |
| `unmappedTierNames` | untiered test, or a tier the hub does not declare | missed (finding 3) |
| `caveats` | package with no map: `UnseenTests` is nil | correct |
| `caveats` | worktree map wins: its Unseen is used | correct |
| `indexVersion` 2 | v1 index read: treated as absent, rebuilt in memory | correct |
| `indexVersion` 2 | `UnseenTest` JSON round-trip with omitempty fields: equal | correct |
| `MapFormatVersion` 3 | written only, never read | inert (finding 1) |
| Judges (`JudgeMap` / `JudgeAgainst`, redundancy kept pool, mutation `planPackage`, `redundant` covers) | unresolved entry that now holds blocks | correct: `notJudgedReason` and `Measured` exclude it, so its blocks never count as cover |

**Findings**

1. **MEDIUM**, class `design`, at `internal/testcover/coveragemap.go:22`. Bumping `MapFormatVersion` to 3 changes nothing a reader does.
   - No reader or refresh path checks `Format.Version`. `testreach.Fresh` (`internal/testreach/fresh.go:26-50`) compares file hashes only, and `reachStep.staleLinked` (`internal/gatecli/reach.go:353-362`) and `freshMaps` (`internal/gatecli/mutate.go:343-350`) use `Fresh`.
   - So a worktree map that v2 wrote, still fresh by hash, is never refreshed. Its unresolved entries stay blockless and the reported miss persists.
   - The shared map for `internal/fabricengine` is v2 and holds `TestRemoveLocal` blockless (confirmed in context). Those entries are captioned "they call what the spawn scan cannot resolve", when the actual reason is that the map run never ran them.
   - Fix: treat a map whose `Format` is nil or older than `MapFormatVersion` as not fresh in the refresh decision. Alternatively, have the caveat name a map older than the format as the cause.

2. **LOW**, class `design`, at `internal/gatecli/reach.go:515-523` (`unseenCause`). An unresolved test whose run passed but covered nothing gets the caveat "the map run recorded their runs alone as covered nothing, with no coverage".
   - `mayRunUnseen` (`internal/testreach/reader.go:166-167`) lists the test only because of its unresolved mark. A no-coverage test with no mark is deliberately left out.
   - So the caveat names a cause that by itself puts nothing in the list, and leaves out the real one.
   - The text is also redundant: "covered nothing, with no coverage".
   - Fix: give `OutcomeNoCoverage` the unresolved cause, keeping the outcome cause for failed, timed-out and skipped runs.

3. **LOW**, class `design`, at `internal/gatecli/reach.go:482-495` (`unmappedTierNames`). Two kinds of test go into no map but get no caveat:
   - A test with no `tiergate.Require` comes back from `ScanTestFuncs` as `Tier 0, Tiered false` (`internal/testcover/testfuncs.go:28`), and `0` is in `mappedTiers`.
   - A tier0 or tier1 test whose tier the hub does not declare is also never mapped (`internal/gatecli/testmapplan.go:21`).
   - Both can call the function with no map holding them. The same gap existed in the old `aboveMapped` check, but the rewrite keeps it.
   - Fix: count a test as unmapped when `!fn.Tiered` or its tier is not declared, the same filter the map run uses.

No swallowed errors and no inconsistent state on failure paths: scan and stale-linker errors still reach `report.Errors`, and a `writeFailed` keep still carries the old entries with a failure marker.

Tool calls:
1. `grep` of `ScanTestFuncs` and `mappedTiers`: the tier value of an untiered test was not in context.
2. `grep` for where `ScanTestFuncs` sets `Tier`: to find the definition the first grep missed.
3. `sed` of `testfuncs.go` and the `mappedTests` filter: to confirm `Tiered` semantics and the map run's tier filter.
