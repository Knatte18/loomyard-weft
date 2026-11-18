# Fix redundant check on unbuildable mutants and foreign-platform tests

`lym gate redundant internal/boardengine` failed after 5 s because one mutant did not compile,
and a full `lym gate test --map --all` on Linux exited 1 over tests that only run on Windows.
This change fixes both.

## What changes

- **Literal mutants keep their type** (`internal/testcover/mutant.go`).
  An integer or rune constant `c` used to become `lymMutantInt(c, id)`, which returns an `int`,
  so `digits[i] == '0'` with a `byte` operand no longer compiled.
  It now becomes `(c + 1>>lymMutantStep(id))`.
  The shift count is not constant, so the untyped `1` takes the type `c` has in its context, and a byte or rune stays one.
- **An unbuildable mutant is dropped, never a failure** (`internal/testcover/mutant.go`, `internal/gatecli/redundantmutants.go`).
  `UnbuildableMutants` blames, in order, the mutants spanning a line the compiler names,
  the mutants of the files it names, then every mutant left, since the package builds unmutated.
  Each round drops at least one mutant, so the rebuilds end, and the dropped mutants count as unbuildable.
  `mutantBuildRounds` is no longer a cap: from that build on, a refused build drops whole files at once.
  `compileErrorLine` also matches type-checker errors that name no column.
- **Platform-only tests are not failed tests** (`internal/testcover/testfuncs.go`).
  `ScanTestFuncs` reads only the files the default build context takes, so a test only another platform compiles is no mapped test.
  A build tag set through `GOFLAGS` is not read.

## Design decisions

- A constant inside an expression built from integer and rune constants alone is not mutated.
  The shift would make the expression non-constant, and an intermediate value could overflow its type with no mutant on.
- An embedded-file line mutant now changes the value after the first `: ` or `=` and keeps the key,
  so a loader that refuses an unknown key does not make every test that loads the file kill the mutant.
  This is a separate behaviour change of `change embedded line`, made while fixing the operator.
- `internal/gatecli/testmap_integration_test.go`: `TestGateTestMapLinkedPackages` no longer puts a `tier1` build tag on its fixture,
  since the scan now honours build constraints.

## To check by hand

- Not run: `lym gate redundant internal/boardengine`.
  The task's done-when asks it to finish and to state which of the five false `boardengine` candidates
  from `docs/research/speed/coverage-map-pilot-2026-11-12/report.md` are gone.
  No run or regression check backs that claim here, so none is made; run it and compare against the pilot report.
- Not run: a full `lym gate test --map --all` on Linux, to confirm it now exits 0.
- `internal/testcover/mutantbuild_integration_test.go`: the test that exercises the whole-file switch at round `mutantBuildRounds`
  does not pin the switch itself; check the switch by hand.
- `internal/testcover/mutant.go`: the exemption for constant-integer expressions is a syntactic check,
  so check that it leaves out no expression whose constant value the shift would change.
