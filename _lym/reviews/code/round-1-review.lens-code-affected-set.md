```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I traced everything outside the touched lines that reads, calls or describes it.

**Per change: the set found, and whether it is covered**

- **`mapPackage` runs unresolved tests, and their entries carry coverage beside the verdict.** The set:
  - `fillMapTests` and `writeFailed` carry-over: covered, since kept entries keep their `Spawn`.
  - `JudgeMap` and `notJudgedReason` (`internal/testcover/verdict.go:125,163`): covered, since an unresolved run is still not judged.
  - `gate redundant`'s `covers` (`internal/gatecli/redundant.go:208`) and the mutation pool (`redundantmutants.go:168`): covered, since both are filtered by verdict reason.
  - `judgeRuns` kept set (`internal/gatecli/redundancy.go:609`) and `planPackage` (`internal/gatecli/mutatecheck.go:235`): covered through `Measured()`.
  - `PossibleKillers` (`internal/gatecli/mutatecheck.go:487`): now also lists cross-package unresolved tests; it is advisory, so covered.
  - `measureTests` `failed_tests` and exit code: these now include unresolved tests' failures, and that follows from running them; covered.
  - `TestGateTestMapLinkedPackages` fixture: updated.
  - `redundancy_integration_test.go:178` fixture comment: still true, because its entry holds blocks and is no cover.
  - The `judgeRuns` doc comment (`internal/gatecli/redundancy.go:587`): **not covered**, see F1.
  - `internal/gatecli/doc.go:804-805` and `pattern/PATTERN-test-speed.md:298-299`: updated.
  - `docs/research/speed/testmap-upkeep-2026-11-11/proposal-opus.md:162` says unresolved tests are never run. It is a dated research record, so no finding.
- **`MapTest.HoldsCoverage` is added, and the `MapTest.Spawn` and `Measured` docs change.** Its one caller is `buildTable`. The testcover `doc.go` passage is updated. Covered.
- **`MapFormatVersion` goes from 2 to 3.** Readers: none act on the version. It is written only at `testmappackage.go:733` and asserted only through the constant at `testmap_integration_test.go:1086,1773`. Covered as far as the affected set goes; the handler owns whether a version nobody reads has any effect.
- **`SpawnMarked` is replaced by `UnseenTests`, and `packageInfo.SpawnMarked` by `Unseen`.** No caller of the old symbol or key `spawn_marked` is left. `reach.go:458` is the one caller of the new one. `index.go` has `indexVersion` at 2. `testreach` `doc.go` is updated. Covered.
- **The caveat text changes.** Readers:
  - `ReachReport.Caveats` rendering (`reachreport.go:261`): text-agnostic.
  - `reachreport_test.go:62,69`: uses its own literal strings.
  - `reach_integration_test.go:209`: checks only that caveats exist.
  - The `gatecli` `doc.go` reach passage: updated.
  - The reach verb doc (`doc.go:443`) does not describe the caveats.
  - The bound line at `pattern/PATTERN-test-speed.md:340` is still true.

  Covered.
- **Rules and commit messages.**
  - Rules: no rule in `PATTERN.md` or the `pattern/` files, `PATTERN-coverage-map-reader` included, is contradicted.
  - Declared PATTERN changes: the directive declares none.
  - Commits: there is no `Pattern-Departure:` or `Loosened-Check:` trailer. The `TestReader` row changed what it expects (an unresolved entry now reaches its block), which is a behaviour change, not a looser check.
  - `Root-Cause:` is present on `94bf82d0`.

**Findings**

- **F1.** Severity: LOW. Class: consistency.
  - Location: `internal/gatecli/redundancy.go:587`.
  - Summary: the `judgeRuns` doc says "An outcome entry and a spawn-marked one record no coverage, so they cover nothing". After this diff, an entry marked unresolved records coverage. It covers nothing only because `judgeRuns` filters on `Measured()` (line 609). The comment should give that reason instead of claiming the entry has no coverage.
  - Evidence: `internal/gatecli/redundancy.go:587,609`; `internal/testcover/coveragemap.go:100-103`; `internal/gatecli/testmappackage.go:129,645`.

Tool calls:
1. Grep across the repo for the old caveat text, "spawn-marked", `SpawnMarked` and `spawn_marked`: the manifest did not cover comments and docs outside the diff.
2. Read `internal/gatecli/redundancy.go:578-592`, `redundancy_integration_test.go:170-190`, and grep for caveat readers: the first grep pointed there.
3. Grep for `TestsReaching` consumers in mutate and the reach verb docs: the manifest lacked them.
4. Read the reach verb section of `internal/gatecli/doc.go:440-460` and `mutatecheck.go` `survivor`, and grep the callers of `caveats` and `UnseenTests`: needed to confirm the reach-answer consumers.
