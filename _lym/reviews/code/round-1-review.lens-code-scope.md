```yaml
lens: code-scope
outcome: reported
```

**Decisions** (record sections other than `## Goal` are empty; no `Added after Discussion` entries):
- F1 — literal operator keeps the type of the literal it mutates, or skips one whose type it cannot keep.
- F2 — an unbuildable mutant is dropped and counted unbuildable, never failing the whole check.
- F3 — a test skipped or unrunnable on the host platform is not a failed test; clean map build exits 0.
- DW (Done when) — `lym gate redundant internal/boardengine` finishes; the five false `boardengine` candidates no longer listed; change description states which are gone.

**Per change → decision:**
- C1 `(c + 1>>lymMutantStep(id))` rewrite (`mutant.go` stepHelper, mutantHelpers step func, opening/closing, literalHelper; `testcover/doc.go` sentence; `TestMutantPlan_WriteOverlay` row; `TestMutantBinary_KeepsTheTypeOfAByteContextLiteral`) → F1.
- C2 column-optional `compileErrorLine` (`runner.go`, `testmain_test.go` standInCompileErrors, `runner_test.go` row) → F2 (the boardengine error named no column, which is why the drop rounds failed).
- C3 `UnbuildableMutants` line→file→all blame with `wholeFiles`; `buildMutants` drops the `round == mutantBuildRounds` failure; mutantBuildRounds comment; `gatecli/doc.go` and `testcover/doc.go` sentences; `TestMutantPlan_UnbuildableMutants` rows → F2.
- C4 `redundant_integration_test.go` `Timeout` fixture → `time.Duration(5 * float64(time.Second))` plus comment → forced by C1 (`5 * time.Second` now compiles mutated, so the fixture needed a still-unbuildable site).
- C5 `ScanTestFuncs` reads only `build.Default`-matching files; `testfuncs.go` comment; `testcover/doc.go` and `gatecli/doc.go` sentences; `TestScanTestFuncs` `d_test.go` row → F3.
- C6 `testmap_integration_test.go` drops `//go:build tier1` from the fixture's `spawn_tier1_test.go` → forced by C5 (the tagged file's tests would leave the scan, as commit `466dc6a3` states); also agrees with PATTERN-test-speed's no-tier-tag rule.
- C7 embedded key-value line mutant marks the value, not the key (`lymMutantLine`, `OperatorChangeEmbeddedLine` comment, `testcover/doc.go` sentences, `TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine`) → DW. With F1–F3 alone, `TestConfigTemplate_ResolvesToDefaults` stayed a false candidate (commit `3a786793`, `summary.md` "Result" section), so this change is required to meet DW; not unrequested scope.

**Per decision → changes:**
- F1 → C1. (A literal whose type cannot be kept, e.g. an integer in a float context, is not skipped at plan time but dropped at build via C3 — within F1's "or skips" together with F2; not a scope gap.)
- F2 → C2, C3.
- F3 → C5 (+ C6 forced).
- DW → C1, C2, C3, C7, plus `_lym/landing/summary.md` "Result on `internal/boardengine`" stating the run finished and listing all five candidates as gone with covering tests and unique kills. The entry names the 2026-11-12 pilot as the source of the five candidates; summary.md uses `redundant-candidates-check-2026-11-17` and says why. I verified: `docs/research/speed/coverage-map-pilot-2026-11-12/report.md` contains no "boardengine" at all (grep count 0), and `redundant-candidates-check-2026-11-17/report.md:28-36` lists the five read candidates matching summary.md's table. So the answer key's citation is wrong (a finding against the answer key, which the rubric forbids); the diff/summary follow the entry's evident intent and disclose the substitution.

**Findings:** none.

Tool calls:
1. `ls docs/research/speed` + grep of the 2026-11-12 pilot and 2026-11-17 check reports for `boardengine`/candidates — context lacked the Done-when's cited source report needed to judge the summary's substitution.
