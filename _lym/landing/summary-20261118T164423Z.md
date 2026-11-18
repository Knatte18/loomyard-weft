# Make lym gate redundant finish on boardengine; map builds exit 0

`lym gate redundant internal/boardengine` failed after 5 s, and a full `lym gate test --map --all` on Linux exited 1 although nothing failed.
Both were found by `docs/research/speed/coverage-map-measurement-2026-11-18/report.md`.
This change fixes both, and the redundant check now refutes all five false `boardengine` candidates.

## What changes

- **Integer and rune constants keep their type.**
  The constant-changing operator rewrites an integer or rune constant `c` as `(c + 1>>lymMutantStep(id))`, not as a generic call returning a rune.
  The untyped 1 of a non-constant shift takes the type `c` takes in its context, so `digits[i] == '0'` against a `byte` still compiles.
- **An unbuildable mutant no longer fails the check.**
  A compiler error line with no column is parsed too.
  `UnbuildableMutants` blames the mutants on a refused line, else those of the refused file, else every mutant left.
  From round `mutantBuildRounds` on, `lym gate redundant` drops whole files instead of failing.
  Only a package that does not build unmutated fails the check.
- **Other-platform tests are not mapped.**
  `ScanTestFuncs` reads only the test files the default build context takes.
  A test in a `//go:build windows` file is no mapped test on Linux, so it never lands in `failed_tests`, and a clean map build exits 0.
- **An embedded key-value line's mutant changes the value, not the key.**
  The embedded-line mutant put its `~` before the key of each line of `boardengine`'s `template.yaml`.
  The board config is loaded strictly against that template, so every such mutant failed every test that loads the config.
  The cover `TestFabricReplay` therefore killed every mutant that `TestConfigTemplate_ResolvesToDefaults` killed, though it checks none of the template's values.
  On a line whose first `: ` or `=` has text after it, the mutant now marks that value and keeps the key.
  Any other line is still marked after its indent.

`internal/testcover/doc.go` and `internal/gatecli/doc.go` state the new rules.

## Result on `internal/boardengine`

`lym gate redundant internal/boardengine` was run from this branch's `cmd/lym`, on the map built at `8040985b`; `boardengine` holds the same files at the branch head.
It finished in 24 min 34 s, at a 2.1 GB peak resident size: 1814 mutants, 0 unbuildable, 41047 runs.
It lists no candidate; 37 tests are refuted, 6 unconfirmed, 5 not judged and 8 kept.

All five false candidates are gone, each now refuted by mutants only it kills:

| Test | Covering tests | Unique kills |
|---|---|---|
| `TestConfigTemplate_ResolvesToDefaults` | `TestFabricReplay` | 3 embedded-line mutants of `template.yaml`, used at `template.go:16` |
| `TestRunHolds` | `TestComputeLayers` | 3 constants at `runlock.go:44`, 1 deleted clause at `runlock.go:61` |
| `TestRunLock_BeamKindAndBeamChange` | `TestRunLock` | 1 deleted clause at `runlock.go:180` |
| `TestRunLock_SketchExemptionIsTheWholeNetChange` | `TestFabricReplay`, `TestMakeSketch`, `TestCheckRunLocks_FamilyProject` | deleted clauses at `sketch.go:83` and `sketch.go:84` |
| `TestReadRaddleStore` | `TestSlugRule_RaddleStore` | 1 constant at `raddlestore.go:150` |

The run with only the first three fixes still listed `TestConfigTemplate_ResolvesToDefaults` as the one candidate, which led to the embedded-line change.

## Design decisions

- The five candidates are the ones `9ce7233b` names, from `docs/research/speed/redundant-candidates-check-2026-11-17/report.md`.
  The task names the 2026-11-12 pilot report, which covers `internal/adoptcli` and lists no `boardengine` candidate.
- The embedded-line rule looks for `: ` and `=` in any embedded file, not only YAML.
  A prose line holding either is still changed, only further along the line.
  A key with nothing after its separator, such as `types:`, is marked at its start as before.

## Tests

- `TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine` (tier 1) builds a real mutant binary over an embedded file.
  It checks the text each mutant yields for a colon line, an equals line, a bare key and a plain indented line.
  It shares a new fixture builder, `buildFixtureMutants`, with `TestMutantBinary_KeepsTheTypeOfAByteContextLiteral`.
