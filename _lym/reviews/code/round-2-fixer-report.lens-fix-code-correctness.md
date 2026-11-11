```yaml
lens: fix-code-correctness
outcome: reported
```

Task restated: check the F1, F2 and F3 changes through the code-correctness lens.

Findings:

- **LOW** — `internal/gatecli/testmap_integration_test.go`, row `ResumedPackageIsBuiltAndMeasuredOnce`. It asserts `Built == ["yielding"]` and exactly one `Packages` entry. It never asserts that the package ran two jobs, and neither does `TestGateTestMapPassYields`. If the retake's `planLockedPackage` (`testmapplan.go:65`) returned `queued=false`, only one job would run. The row would then pass with the F3 defect in place. The replan most likely queues the missing `TestYieldingHangs`, since the map holds only `TestYieldingAOne` and the resumed job re-runs the hanging test. I could not confirm this without running the row on the pre-fix code. A cheap guard is to also assert that `TimedOut` equals `["yielding.TestYieldingHangs"]` and that the measure's `Tests` is at least 2. A resumed job would show both. A single job that yielded would not.

- **LOW** — `internal/gatecli/testmapaudit.go:64` (F1). The review asked for a coverage row where HEAD moves between the last queued job and the audit, and none was added. The only place a landing could be injected is between jobs, which the stand-in test harness cannot do deterministically. The fixer should list this as deferred, with that reason, in the fixer-report.

- **LOW** — `internal/gatecli/testmap_integration_test.go`, the `!handCall` branch of `runPassYields`. Its early `return` skips the `ReadExitRecord` assertion at the end of the function. That row therefore never checks the pass's exit record.

The rest of the change set holds up:

- **F1:** `r.readHead()` returns its error, and `headMoved` is then tested as before. `audit` is called only from `mapPackages` after `r.head` is set, so the read is valid.
- **F2:** `refuseTrackedTree` accepts the path `<bolt>/.lym/testmap-upkeep.lock`. The `.lym` segment resets the Bolt check, and its next segment is not `locks`, so it returns nil. The `defer` releases the lock on every return path, including the early refusals. A crashed holder frees the flock through the OS. Same-process callers in tests block only until the first call returns, so there is no deadlock.
- **F3:** `addMeasure` is called only from the deferred append in `mapPackage`. The `built` guard changes `write` callers not at all. A package built in a queued job and then audited in the same call now yields one merged measure and one `built` entry. That matches the finding's intent.

Further tool calls (1 of 6): `grep` of `planLockedPackage` in `internal/gatecli/testmapplan.go`, because the file was not in my context and I needed to know whether a retake always queues.
