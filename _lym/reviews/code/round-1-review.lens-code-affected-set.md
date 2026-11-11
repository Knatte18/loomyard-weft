```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, this lists the set I found, then the findings.

**Set per change**
- `test_map_max_age_days` retired from gateslot (`Config`, `configFile`, `template.yaml`, `RetiredKeys`, `ConfigSpec` category, `decodeConfig`). Covered.
  - The `config_test.go` row for the retired key loads, and the old "zero" refusal row is removed.
  - gateslot `doc.go` and the template are updated.
  - No other reader of `TestMapMaxAgeDays` exists outside the research docs.
  - The `hubreconcile` fixture `commitRetiredKeys` seeds only darn, gate and landing keys that are not the age key. That is fine: no assertion depends on it.
- `ExitRecord.Unbuilt` replaced. Covered.
  - The only production reader and writer is `testMapRun`.
  - The `lymdirs` comment, `testcover` `doc.go` and the gatecli `doc.go` line are updated.
  - A test reads an old record that holds `unbuilt`.
  - Nothing in the dobby reads the record (finding F1).
- `measured()` moved to `MapTest.Measured`. Covered. The selector and the redundancy step use it, and the redundancy comments are updated.
- `Map.Failure` and `FailedAt` (D5). Covered by `TestMapFailedAt`, the planner and the audit. The redundancy step's `mappedPackages` doc is updated.
- Dobby refresh wording (`dobbyengine`). Partly covered (finding F2).
  - The dobby `doc.go` is updated.
  - `testmap.go`, `config.go` and `template.yaml` still describe the old "stale packages" behaviour.
- `PATTERN-test-speed.md` is updated and consistent with the new behaviour.
- Commit messages: no `Pattern-Departure:` trailer is present, and none is needed.
- Declared PATTERN changes: the constraints list no PATTERN add, change or remove.

**Findings**

F1. MEDIUM, class `scope`. `internal/dobbyengine/testmap.go` (whole step) and `internal/gatecli/testmap.go:228-247`.
- Acceptance D12 ends "and the dobby reads it". The diff makes no dobby read of the exit record. `grep ReadExitRecord` finds readers only in gatecli tests.
- `_lym/landing/summary.md` admits this and defers it to D8. The decision record lists the dobby read in D12's acceptance, so the gap still stands.
- The summary's "Not in this change" defers a D12 acceptance line. If the parent accepts that, record it with `lym loom decision add`.

F2. LOW, class `consistency`. `internal/dobbyengine/testmap.go:13`, `internal/dobbyengine/config.go:42`, `internal/dobbyengine/template.yaml:5`.
- All three still say the refresh "builds at most `test_map_batch` stale packages" and that it builds "a coverage map".
- The behaviour changed to plan every map and measure the queued ones, and the diff already corrected the dobby `doc.go`.
- Fix: reword the three to the dobby `doc.go` wording at line 491, "measures at most `test_map_batch` of the packages it queues".

F3. MEDIUM, class `decision`. `internal/dobbyengine/config.go:64-65` and `internal/dobbyengine/template.yaml:4-5`.
- The record's `PATTERN-config-layers` constraint says `test_map_batch` and `test_map_every` leave their templates as retired keys, with `test_map_every`'s 0 carried by D8's migration.
- The diff retires neither key. Both stay live in `dobbyengine`: `Config`, `ConfigSpec`, the template and the tests.
- The record's Delivers list names only `test_map_max_age_days` as retired, and the dobby still starts the refresh by landing count. So the retirement may belong with D8.
- Fix: the parent decides whether the constraint binds this diff. Retiring the keys here would break the dobby's count-driven start, so recommend deferring to D8 and recording that with `lym loom decision add`.

**Tool calls**
1. A grep of the repo for `TestMapMaxAgeDays`, `test_map_max_age_days` and `unbuilt`: my context lacked readers outside the diff.
2. A grep for `RetiredKeys`, `test_map_batch` and `test_map_every`: to check how the dobby keys are handled.
3. A grep for `ReadExitRecord` and `ExitRecord` outside `testcover`: to find readers, including the dobby.
4. A read of `ensure_integration_test.go` lines 248-290, `dobbyengine/testmap.go` lines 8-22 and a grep for "stale" in the dobby and gatecli docs: to check the reconcile fixture and stale wording.
