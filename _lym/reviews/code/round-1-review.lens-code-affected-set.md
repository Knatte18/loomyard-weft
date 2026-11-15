```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I list what reads or describes it and whether the diff left that consistent.

**Per change: the set found, covered or not**

1. **Removed `lym gate testmap --upkeep`, `runproc.RoleTestMap`, the exit record and `lymdirs.TestMapExitFileName`.**
   - Readers: beamcli (`startTestMap`, `testMapRunning`, seams), dobbyengine (`testMapStep`), orchcli `beamDown`/`endTestMap`, fabriccli `liveUnsparedChild`/`dobbyPassInFlight`.
   - Docs: runproc/doc.go, the beamcli, orchcli, fabriccli and dobbyengine doc.go files, docs/overview.md, README.md, pattern/PATTERN-test-speed.md.
   - Tests: beamcli, orchcli, fabriccli and dobbyengine tests.
   - Covered: a repo-wide grep finds nothing left outside docs/research.
2. **Retired `test_map_upkeep` key and removed beam `MigrateConfig`.**
   - Set: `configreg` Modules entry, configsync_test, configreg runsettings_test, dobbyengine `ConfigSpec` RetiredKeys, template.yaml, doc.go and config_test.
   - Covered.
3. **Retired gate scope `coverage` (`gateslot.ScopeCoverage` becomes `retiredScopeCoverage`, migration and load refusal).**
   - Set: gateslot template.yaml, doc.go, tiers.go, config.go and config_test; gatecli `withGateScope`, `computeScope`, `planTierRuns`, cli.go `testRequest` fields, run.go, the doc.go Coverage scope section and template block.
   - Also: scopes_integration_test, loomcli landingdeps_verify_test, and the PATTERN-test-speed gate-key table.
   - Covered: no `scope: coverage` is left in contracts, stencils or docs; the grep found none.
4. **Removed the `selected` scope form and `selectedTestsRun`.**
   - Set: tiers.go, tiers_test (`TestPlanTierRuns`, `TestTierNumbers`), run.go's `tierNumbers` comment.
   - Covered. The remaining "selected" wording in gatecli/doc.go:187–272 belongs to the dotnet share and is unrelated.
5. **Removed `gateslot.Pool.TryAcquireSpare` and `Pool.Contended`.**
   - Set: pool.go, pool_test, gateslot doc.go, gatecli testmapyield.go (deleted) and testmapaudit.go.
   - Covered.
6. **`testmapyield.go` renamed to `testmapwait.go`.**
   - Set: the cmd/lym/daemonwakeups_test.go `unboundedWaitFiles` list and the PATTERN-daemon-wakeups.md list.
   - Covered.
7. **Removed `testcover.TestFunc.Parallel` (`declaresParallel`, `requireKitName`).**
   - Set: gatecli `nextRound` (deleted), testfuncs_test, testcover doc.go.
   - Covered.
8. **`FuncIndex.CoveredFuncs`/`ProfiledFuncs` now take a package's own blocks; `ProfileByPackage` added.**
   - Callers: gatecli testmappackage.go `reduce` and `newBlockTable`; the grep shows no other production caller of `CoveredFuncs`/`ProfiledFuncs`.
   - Doc: testcover doc.go. Test: blockkey_test.
   - Covered.
9. **`testcover.PackageMeasure` moved to `gatecli.testMapMeasure`; `PlanCounts` moved to upkeep.go.**
   - Readers: testmap.go output and the integration test decode.
   - Covered.
10. **`Miss.String` moved to `miss_string_test.go`.**
    - Production callers: none left; the coverage scope's label was the only one.
    - Covered.
11. **The coverage scope was the sole reader of `impactset.ModuleTargets.Importers`/`BinaryBuilders`.**
    - Set: impactset/targets.go:45–50 and 147–152, graph.go:148–172 (`splitUnchanged`, `packageArgsOf`), impactset/doc.go:36–37, docs/overview.md's impactset line.
    - **Not covered**: production still computes and documents these fields, but nothing in production reads them (finding 1).
12. **Declared PATTERN changes.** The directive declares none, so there is nothing to check.
13. **Commit messages and departures.** No `Pattern-Departure:` trailer is in the range. The bodies of commits 6b413073 and 09ff8a58 match the diff.

**Findings**

1. LOW · consistency · `internal/impactset/targets.go:45-50,147-152`, `internal/impactset/graph.go:148-172`, `internal/impactset/doc.go:36-37`
   - Summary: removing the coverage scope left `ModuleTargets.Importers`/`BinaryBuilders` and `splitUnchanged` with no production reader. They are still computed on every `diff` scope derivation and documented as if something used them.
   - Evidence: the only reader is `internal/impactset/targets_test.go:23`, from a grep of `.Importers`/`.BinaryBuilders`.
   - The fields came in with cdc3522a ("gate: add a coverage scope that narrows importing packages' tests").
   - The removal the board entry asks for ("the gate's map-based selection") leaves them as dead data, against code-quality's "If something is unused, delete it".
   - Fix: delete the two fields, `splitUnchanged`, the `Importers`/`BinaryBuilders` assignments and their doc lines, and fold the `targets_test` row.

Tool calls:
1. Repo-wide grep for upkeep, coverage-scope, exit-record, spare-slot and `selected` leftovers in every text file (contracts, stencils, docs): the manifest's grep covered only .go/.md/.yaml.
2. impactset/doc.go coverage mentions, gatecli doc "selected" wording, the test-only-code allowlist and the hub gate.yaml: the manifest had not read these.
3. Working-tree selector paths in testcover and the impactset `splitUnchanged`/`packageArgsOf` definitions: needed to confirm finding 1 and its full extent.
