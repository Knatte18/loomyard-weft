```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches. It found two LOW consistency defects and nothing else left wrong.

**Per change, the affected set and whether the diff covers it:**
- **`createKillOnCloseJob(name, memoryLimit)` signature:** called from `namedjob_windows.go:63`, `tied_windows.go:15` and `memorycap_windows.go:15`. All three are updated, and no other caller exists. Covered.
- **New `redundancy_test_memory_mb` key:**
  - `gateslot`: `Config`, `configFile`, the `ConfigSpec` category (cross-pair, like its siblings), `decodeConfig` validation, the accessor, `template.yaml`, `doc.go`, and `config_test.go`'s `templateDefaults` plus its zero row. Covered.
  - Readers: `gatecli` `redundant.go`, `redundancy.go`, `testmap.go` and `testmappackage.go` all read it.
  - Help and docs: the `redundant` long help, `gatecli/doc.go` (redundancy report step, testmap and redundant) and `docs/overview.md` (redundant and testmap). Covered.
  - Hub-config rules: no reinterpretation is needed for a new key under PATTERN-config-layers, and `LocalKeys` is derived. Covered.
  - Wording gap: the template comment's wording for the redundancy report step is wrong (finding 2).
- **`RunResult.MemoryCapped`:**
  - Mutation check (`record`/`runJobs`): covered.
  - Redundancy report step: reason `reasonMemoryCapped`. Covered.
  - Testmap: `testmappackage.go:675-685` falls through to `OutcomeFailed`, which matches the doc's "recorded as failed". Covered.
  - `cmd/testtiming`: a failed action. Not read further.
- **Mutant timeout floor lowered from 10 to 2:** the doc names it by symbol (`gatecli/doc.go:1146`). The test comment at `redundant_integration_test.go:197-200` still holds: 2 × 2 s = 4 s, the same floor as before (10 × 0.4 s). Covered.
- **`mutationStats` gains `timed_out` and `memory_capped`:** the JSON readers are `redundant_integration_test.go`, which asserts both lists, and `gatecli/doc.go:1176-1178`. The research data files under `docs/research` are measurement records and are exempt. Covered.
- **Constants `selfCgroupFile` and `cgroupMount` replace repeated literals in `proc`:** every use inside `proc` goes through them. `cmd/testtiming/resources.go:68` has its own `cgroupRoot`, which predates this diff and sits outside the range. Covered.
- **Spawn-observability exemption for `internal/proc/memorycap_linux.go`:** the stated reason is wrong for the probe spawn (finding 1).
- **`redundant` long help shortened (commit 7ef11dcc):** the dropped clause about where the map comes from survives in `gatecli/doc.go:1128` and `docs/overview.md:943`, and the remaining help is still true. Covered.
- **Declared PATTERN changes:** the directive declares none. Nothing to land.
- **Commit messages and departures:** no `Pattern-Departure:` trailers exist. Every commit message matches its diff.
- **Stencil wording:** the diff changes no stencil. Not applicable.

**Findings:**

1. **LOW, consistency.** In `cmd/lym/spawnobservability_test.go:64-68`, together with `internal/proc/doc.go:63` and `internal/proc/memorycap_linux.go:50`, the exemption's reason says "the callers of RunMemoryCapped log the command and its end". But `probeMemoryScope` spawns `systemd-run … cat /proc/self/cgroup` itself (`memorycap_linux.go:50`), and no caller logs that spawn or its end.
   - `memoryCappedParallel` (`redundantmutants.go`, `logger.Info` after `proc.MemoryCapCoversTree()`) logs only what the probe concluded.
   - `testcover.RunAlone` logs the test binary, not the probe that `hostMemoryScope()` triggers.
   - Fix: make the written reason, and the `proc` doc sentence, true for the probe. Either have the probe's first callers log that it ran, or state in the reason that the probe spawn goes unlogged.

2. **LOW, consistency.** `internal/gateslot/template.yaml:6` says a test that passes `redundancy_test_memory_mb` "then counts as failed". The redundancy report step does not count such a test as failed: it leaves the test out as not judged, with `reasonMemoryCapped` (`internal/gatecli/redundancy.go:553-554`). `gatecli/doc.go:590-591` describes that correctly. The comment on `redundancy_test_timeout_sec` one line above words the same case as "the test is left out". Fix: reword the memory key's comment to match how each run treats a capped test.

**Tool calls:**
1. Searched every file naming `redundancy_test_timeout_sec`, the callers of `createKillOnCloseJob`, readers of the mutation answer's JSON, mentions of the timeout floor and the cgroup path literals. The context held only the diff, not the rest of the repository.
2. Searched the gatecli help and comments for the config keys, PATTERN mentions of the redundancy report, the test's floor comment and the `proc` constant users. I needed to know which unchanged files describe the changed behaviour.
3. Read PATTERN-test-speed's redundancy section and searched for kill-on-close references, to check whether the pattern describes the bounds the diff changes.
4. Checked whether `gatecli/doc.go` still states the map-source clause that commit 7ef11dcc dropped from the help, and read that commit's stat.
5. Searched `gatecli/doc.go` for redundant's map-reading sentence, since the previous search missed it.
6. Read `testMapRun.runTest`'s outcome switch and searched the overview for the report step's bounds, to confirm how testmap records a capped run.
