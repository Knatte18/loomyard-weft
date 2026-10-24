# Fixer-report — code round 1

## Changes

- F1 (MEDIUM, fixed as proposed): `planFileNames` in `internal/websterengine/fingerprint.go` also skips `planparser.IsLensReportFileName`; the doc comments of `fingerprint` and `planFileNames` name the lens reports as records about the plan.
  `TestFingerprint` gained an "ignores a lens report" row, and every no-change row now also asserts `planFileHashes` is unchanged.
- F2 (MEDIUM, fixed as proposed): `TestBurlerGeometry` in `internal/hubgeom/hubgeom_test.go` asserts `SpecsDir` equals `fabricengine.SpecsDir(hub)`; the one in `internal/standalonegeom/standalonegeom_test.go` asserts it equals `SpecsDir(stateDir)`.
- F3 (LOW, fixed as proposed): the cluster row of `TestEngine_Review_TableShape` sets `SpecsDir` in the Geometry and asserts the review instruction files name `stencilstore.Path(specsDir, lensreport.SpecName)`.
- F4 (LOW, fixed as proposed): `burler-template-reviewer.md` is back to its two original lines; `burler-cluster-fan.md` states the lens reports are the one exception to the template's "nothing else" and "no other file".

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the fan part's "rules above" referred to text in the prompt template rather than above it in the same file; fixed as proposed, now "the write-surface rules in your prompt".
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
