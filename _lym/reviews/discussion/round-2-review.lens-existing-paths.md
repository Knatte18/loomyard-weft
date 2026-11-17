```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state the record changes, trace the existing code paths that write or read it and say whether the decision survives them.

**Paths traced, per decision**

- **D1, detached keying and replay** (`detachfront.go:370-391`): the key hashes the verb, the run directory and the words built from the request's fields. Adding `--map` to those words separates map runs from plain runs. **Survives.**
- **D1, fixer and lints on a `--card` call**: the gate-key `fixer` and lints belong to gate keys (`gateslot/doc.go:170-175`). D1's Bound says map mode runs no lint or card fixer. **Survives.**
- **D3, `LYM_PREBUILT_LYM`**: its only reader is `lymbin.prebuilt` (`lymbin.go:49-59`). The map loop skips `OutOfProcess || Unresolved` tests (`testmappackage.go:139`), and `spawnRef` marks every lymbin reference (`spawnscan.go:355`). No run test can read the variable. **Survives.**
- **D3, tier `env`** (`renderTierEnv`, `testmappackage.go:670`): the tier's `GOFLAGS` `-p`/`-timeout` value is inert for a binary that runs directly. **Survives.**
- **D4, `tier_parallel` override**: it reaches both `Limits.Tier0Parallel` and each tier's `Parallel` (`tiers.go:32-33`, `applyTierParallel`). **Survives.**
- **D4, memory clamp** (`clampParallelToMemory`, `redundantmutants.go:540-548`): this is the existing reader of `redundancy_test_memory_mb`. D4 adds the `memory_mb` quotient on top of it. **Survives.**
- **D4, memory guard** (`guardOwnTests`, `memoryguard.go:282`): the guard samples the calling process's own tree and stands down only under a live ancestor `lym gate test`. This is the same as the verb's behaviour today. **Survives.**
- **D6, dirty-file reader** (`testmaptree.go:79` `dirtyOwnedFile`, `testmappackage.go:177,495`): this check exists today. The module root holds no Go package, and `.lym`/`_lym` are git-ignored, so a run's own files never make a package dirty. **Survives.**
- **D7, slot holder label**: `testMapSite` is read only as a holder label (`testmap.go:38`, `testmappackage.go:591`), and nothing branches on it. **Survives.**
- **D7, green records and run log**: map mode writes none, and D7 states that it subtracts no green tier. **Survives.**
- **D8, map location**: `coverageMapsDir` (`testmap.go:318-329`) is shared with `lym gate redundant`. The report step reads only the hub's shared maps (`redundancy.go:286`). That behaviour is unchanged, and round 1 rejected it as a finding. **Survives.**
- **D9, live verb text in Go code**: the remaining mentions are `redundant.go:104,129,162`, `prebuild.go:1`, `gateslot/doc.go:67,87,177,184` and `dobbyengine/doc.go:506`. They are covered by D9 and Scope. **Survives.**
- **D2/D9, testmap-only config keys**: none remain. `test_map_max_age_days` is already retired (`gateslot/config.go:136,174`). **Survives.**
- **D3/D4/D7, the time bounds the verb reads today**: **Breaks.** See finding 1.

**Findings**

1. **MEDIUM, design. D3, D4 and D7 do not say which time bounds a map run's builds and tests run under.**
   - Today the verb reads `redundancy_build_timeout_min` for each coverage build and `redundancy_test_timeout_sec` for each test.
   - Evidence: `internal/gateslot/doc.go:177-182`, plus `internal/gatecli/testmappackage.go:647` (`r.buildTimeout`) and `:675` (`r.testTimeout`).
   - Under `lym gate test`, a tier carries its own `timeout_min` ("minutes the tier may run once its slots are held") and `test_timeout_min` (`internal/gateslot/tiers.go:34-36`).
   - D1's Bound lists the tier settings map mode skips: commands, `filter`s, `memory_mb` scopes and the shared build. It does not mention `timeout_min` or `test_timeout_min`.
   - D3's "timed-out test" and D7's "build failed" outcomes both depend on a bound the record never names.
   - A plan writer who moves the loop under the gate could apply tier 1's `timeout_min: 30` (`_bolt/_lym/config/gate.yaml`) to a whole `--all` map run, or the 20-minute `test_timeout_min` to each test. Either reverses today's bounds, and the record says nothing about it.
   - Fix: state in D3/D4 that map mode keeps the `redundancy_build_timeout_min` and `redundancy_test_timeout_sec` bounds and does not apply the tiers' `timeout_min` or `test_timeout_min`, or make the opposite choice explicitly.

**Tool calls**

- Grepped `detachfront.go` keying, map-location readers and `memoryguard.go` functions: the manifest held neither the detached key derivation nor the guard.
- Read `detachKeyDir`, `testMapLocation`/`coverageMapsDir`, `memoryGuardApplies`/`guardOwnTests` and the `Site` readers: I needed their bodies to judge D1, D8 and D4.
- Grepped live "gate testmap" text outside the verb's files: to check what D9 and Scope must cover.
- Read `gateslot/doc.go`'s holder and config paragraphs and the `test_map` config keys: to find settings the verb reads, which led to finding 1.
- Checked for a root Go package, `.gitignore` and the existing dirty-file logic: to judge whether D6's dirty check misfires on a run's own `_lym`/`.lym` files.
