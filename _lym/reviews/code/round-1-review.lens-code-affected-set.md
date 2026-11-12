```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found:**
1. **`test_map_every`, `test_map_batch`, `LandingsSinceTestMap`, `TestMapEvery`, `TestMapBatch`.**
   - A repo-wide grep finds them only in `internal/dobbyengine` (`config.go`, `doc.go`, `config_test.go`), `internal/configreg/runsettings_test.go` and `docs/research/speed/testmap-upkeep-2026-11-11/*`.
   - The `dobbyengine` and `configreg` hits are the migration and its tests; the research docs are a measurement record and exempt.
   - Covered.
2. **`StartTestMap` signature, new seams, `testMapRunning`.**
   - The only wiring is `hubDobbyDeps` in `internal/beamcli/dobbydeps.go`.
   - Tests were updated: `dobbydeps_integration_test.go`, `dobbytestmap_unix_integration_test.go`, `handover_test.go`, `loop_move_test.go`.
   - Covered.
3. **`Spec.MigrateLayer`.**
   - It is applied only in `stackLayers` (`layers.go:609`). The `StackLayers` callers `reviewbudget`, `modelspec`, `batcher` and `runprofile` all set no hook, so they are unchanged.
   - The public `Layers()` (`layers.go:485`) and the held-unit scan at `layers.go:654` read unmigrated layers. Only `held` is built from those layers, and it matters only for a key a layer holds; for a retired key the hook's effect there is immaterial.
   - Covered.
4. **`dobbyPassInFlight` sparing `RoleTestMap`.**
   - Its doc comment and `fabriccli/doc.go` were updated.
   - Covered for docs. The diff changes no `fabriccli` test; the code-tests lens owns that.
5. **PATTERN and docs.**
   - `PATTERN-test-speed.md`, `docs/overview.md`, `dobbyengine/doc.go`, `beamcli/doc.go`, `fabriccli/doc.go`, `configengine/doc.go` and `runproc/record.go` are updated and agree with D8 and D11.
   - The directive declares no PATTERN changes, and no commit message carries a `Pattern-Departure:` trailer.
   - Covered.
6. **`RoleTestMap` consumers in `orchcli`.** Not fully covered (finding F1 below).

**Findings:**

- **LOW, consistency:** `internal/orchcli/doc.go:48-50`, `internal/orchcli/stop.go:37`, `stop.go:50`, `stop.go:74` (error text "end the test-map refresh") and `internal/orchcli/cli.go:125` still call the ended process "the test-map refresh the dobby started".
  - After this diff it is the upkeep pass, which registers itself, and the dobby no longer registers it.
  - The behaviour is still right: `EndRole` over `RoleTestMap` under `HubOwner` ends it, and the process-group note still holds.
  - The wording is stale next to the updated `beamcli/doc.go:328` and `docs/overview.md`.
  - Fix: reword to "the coverage-map upkeep pass" in those places, and update the matching `end the test-map refresh` expectation in `internal/orchcli/beamdown_test.go:3,42` if the error text changes.
  - Evidence: the grep of `test-map refresh` across `internal`, and `orchcli/doc.go:48-50`.

I found no other stale member of the affected set.

**Tool calls (2):**
- One grep for `readLayers(`, `stackLayers(` and `StackLayers(` callers, because the manifest didn't show who else reads layers.
- Two reads of `layers.go` at 478-507 and 645-675, to see whether `Layers()` and the held-unit scan bypass the hook.
