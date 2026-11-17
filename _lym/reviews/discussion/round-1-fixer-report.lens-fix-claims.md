```yaml
lens: fix-claims
outcome: reported
```

I checked every sentence the fix added or rewrote that states a bound, behaviour, count or fact about the code.

**LOW**: D3, "Map mode builds no `lym`: `LYM_PREBUILT_LYM`'s one reader is `internal/testkit/lymbin`, and `testcover.ScanSpawns` marks every test that reaches it as spawn-marked, so no test the run measures reads that variable."
- **Issue:** the claim that every test reaching `lymbin` is spawn-marked does not hold for tier0 files. The scan gives a test in a tier0 file an empty verdict whatever it references, so such a test is not spawn-marked even if it reaches `lymbin`.
- **Evidence:** `internal/testcover/spawnscan.go:172` says "A test in a tier0 file always gets an empty verdict, because a tier0 test spawns nothing".
- **Effect:** the conclusion still holds only because tier0 tests spawn nothing by invariant. The sentence should say that, or narrow the claim to tests above tier0.

The other claims hold against HEAD:
- **D1, a package with no mapped test gets a map of its file hashes:** `testmappackage.go:131-132`.
- **D1, the refused flags `--stop-on-fail` and `--dir`/`-C`, and `--main-packages` with `--card`:** these flags exist (`cli.go:119-130`).
- **D4, the count is 0 when `memory_mb` is below `redundancy_test_memory_mb`:** integer division gives 0.
- **D4, the way forward `lym config gate`:** it can set `tiers.tierN.memory_mb` by its dotted path (`template.yaml:17`) and the top-level `redundancy_test_memory_mb` (`template.yaml:7`).
- **D4 Bound, coverage builds take tier 0's `parallel` as `-p`:** `HolderEnv(..., limits.Tier0Parallel)` at `testmappackage.go:609`.
- **D6, the check runs after the build has listed linked packages and compiled:** `dirtyOwnedFile` runs after `takeBuild`.
- **D6, package file ownership:** `doc.go:1071-1072`.
- **D6 Bound:** the same bound is stated at `doc.go:1133-1134`.
- **D9, the four refusals sharing `rebuild`:** `redundant.go:169,171,173,228`.
- **PATTERN-gate-slot, "a third place":** the background says "in one of two places" (`PATTERN-gate-slot.md:12`).

Tool calls:
- One Bash call reading `internal/gateslot/template.yaml` and grepping `internal/testcover/spawnscan.go` for its tier0 lines. My context did not show how tier keys are set through `lym config gate` or what the scan does for tier0 files.
