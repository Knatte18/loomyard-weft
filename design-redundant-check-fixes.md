# lym gate redundant fails on an uncompilable mutant; a map build exits 1 on Windows-only tests

`redundant-check-fixes` · Task · bug, gate, testkit · running · Preflight

## What was seen

`docs/research/speed/coverage-map-measurement-2026-11-18/report.md` (`122a41f1`):

1. `lym gate redundant internal/boardengine` fails after 5 s:
   `overlay-3/handles.go:62: invalid operation: digits[lymMutantInt(0, 274)] == lymMutantInt('0', 275) (mismatched types byte and rune)`.
   `9ce7233b` ("testcover: mutate data literals and delete boolean-chain operands") rewrites a rune literal as a call returning an `int`,
   so a comparison with a `byte` no longer compiles, and the verb takes that one failed compile of the whole mutant binary as a failure of the check.
   The runs of 2026-11-17 handled 52 unbuildable mutants and finished, so the data-literal operator is the cause.
2. A full `lym gate test --map --all` on Linux exits 1: `failed_tests` lists ten tests that only run on Windows, though nothing failed.

## Fix

- The literal operator keeps the type of the literal it mutates (a rune or byte literal stays one), or skips a literal whose type it cannot keep.
- A mutant that does not compile is dropped and counted as unbuildable, never failing the whole check.
- A test skipped or unrunnable on the host platform is not a failed test, so a clean map build exits 0.

## Done when

`lym gate redundant internal/boardengine` finishes, and the five false `boardengine` candidates from the 2026-11-12 pilot
(`docs/research/speed/coverage-map-pilot-2026-11-12/report.md`) are no longer listed; the run states which are gone in its change description.
