```yaml
lens: code-affected-set
outcome: reported
```

**Per change — the set found, covered or not**

1. **`UnbuildableMutants` gains `wholeFiles`; blames line → file → every mutant left.**
   - Production caller `internal/gatecli/redundantmutants.go:410` — updated.
   - Test `internal/testcover/mutant_test.go` `TestMutantPlan_UnbuildableMutants` — updated (old "a line of another file → none" row became "every mutant left").
   - Docs `internal/testcover/doc.go:58-59`, `internal/gatecli/doc.go:1189-1192` — updated.
   - `Unbuildable` stats doc (`redundantmutants.go:49`, "those dropped for a compile error") — still true (every drop follows a compile error).
   - Covered.
2. **`mutantBuildRounds` meaning changed (no longer a cap; the round from which whole files drop).**
   - Its doc (`redundantmutants.go:35-36`), `buildMutants` doc (`redundantmutants.go:389-394`), `gatecli/doc.go:1189-1190` — updated.
   - No other reference (grep: none outside these).
   - Covered.
3. **`lymMutantInt` helper replaced by `stepHelper`/`lymMutantStep` and the `(c + 1>>…)` rewrite.**
   - `mutant_test.go:360` overlay expectation — updated.
   - `literalHelper`/`scanPackageDecls` (`mutant.go:432-433`) — use the new name.
   - `testcover/doc.go:48` — updated.
   - `redundant_integration_test.go:40,82` comment + fixture — updated so the `Timeout` constant stays unbuildable, keeping the `Mutants != 20 || Unbuildable != 1` assertion (`redundant_integration_test.go:341`) meaningful.
   - Only remaining `lymMutantInt` text: the stand-in compiler error in `testmain_test.go:56`, which reproduces the recorded real output; the parser reads only its file/line, so not stale.
   - Covered.
4. **`compileErrorLine` now matches a column-less line.**
   - Sole reader `BuildMutantBinary` (`runner.go:364`); fixture `testmain_test.go:53-56`; expectation `runner_test.go:141` — updated.
   - Covered.
5. **`lymMutantLine` marks a key-value line's value.**
   - `OperatorChangeEmbeddedLine` doc (`mutant.go:46-48`), `testcover/doc.go:52-53` — updated.
   - `mutant_test.go:362` `lymMutantLine(...)` call text — unaffected (only the helper body changed).
   - New test `mutantbuild_integration_test.go`.
   - Covered.
6. **`ScanTestFuncs` reads only default-build-context test files.**
   - Callers: `gatecli/testmapplan.go:15` (`mappedTests`), `gatecli/redundancy.go:585` (`judgeRuns`).
   - All map builds call `BuildBinary` with empty tags (`testmappackage.go:203`, `redundancy.go:478`, `redundantmutants.go:369`), so a test needing a build tag was never in the binary; dropping it from the scan matches what the binary holds.
   - Doc `testfuncs.go:35-38`, `testcover/doc.go:26`, `gatecli/doc.go:1122` — updated; `testfuncs_test.go` row added.
   - Fixture `testmap_integration_test.go:1637` drops its `//go:build tier1` tag — consistent with PATTERN-test-speed's "No test file's build constraint names a tier tag" (`pattern/PATTERN-test-speed.md:25`).
   - `impactset/tiertests.go:23` has its own "build constraints are not read" comment — a separate scanner, not `ScanTestFuncs`; untouched by this change.
   - PATTERN-test-speed coverage-map/redundant-check text (`pattern/PATTERN-test-speed.md:289-320`) says nothing about platform files or mutant build rounds — nothing stale.
   - Covered.
7. **Repo rules / PATTERN / commit messages / trailers.**
   - Directive declares no PATTERN changes.
   - No commit carries `Pattern-Departure:` or `Loosened-Check:`; none needed — the changed assertions in `mutant_test.go`/`runner_test.go` follow a behaviour change or add an expectation, none loosened.
   - No stencil changed.
   - Each commit message's claims match its diff per the code in context.
   - Covered.

**Findings:** none.

**Tool calls:**
1. grep for stale references (`lymMutantInt`, "Build constraints are not read", "another platform") across code/docs/pattern — context lacked a repo-wide search for leftover text.
2. grep of `gatecli/doc.go` and gatecli prod code for unbuildable counts / mutant helper — context lacked the `Unbuildable` stats field doc.
3. grep of `redundant_integration_test.go` for its unbuildable assertion — context lacked the assertion the `Timeout` fixture change feeds.
4. grep PATTERN files for test-function/build-constraint rules — context lacked which rule docs describe the scan.
5. read `pattern/PATTERN-test-speed.md:285-320` — the coverage-map/redundant-check rules; context had only the file's first 80 lines.
6. read `pattern/PATTERN-test-speed.md:320-345` — rest of redundant-check rules; same gap.
