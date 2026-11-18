---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: The proposed stand-in refuses five builds each naming a line no remaining mutant spans, but a line no mutant spans drops the whole file at round 1, so the cap is never reached. The stand-in instead names lines 13, 18, 26, 27 and 29 of cov.go, each spanned by mutants, so builds 1 to 4 drop by line and build 5 drops the whole file.
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer-report, round 1

## Changes

- F1 (`446cee34`): `fileSites` in `internal/testcover/mutant.go` marks fixed every literal and package-level literal constant of a binary expression built from integer and rune constants alone, through parentheses, unary and nested binary operators, for function bodies and package-level variable values.
  `TestPlanMutants` gains a `Half` fixture function (`return (1 << 63) / 2`) and a row asserting no mutant on it.
- F1 follow-up: the package doc `internal/testcover/doc.go` names the exemption.
- F2 (`356401f1`): `TestGateRedundant` gains a nested subtest that runs `lym gate redundant ./cov` with a stand-in `go`.
  The stand-in is the re-exec'd test binary in the role `helperRoleRedundantGo`. It refuses the first `mutantBuildRounds` mutant builds with a compile error at one line each, then forwards to the real go command.
  The subtest asserts exit 0, no mutant left, every mutant of cov.go unbuildable, and `mutantBuildRounds+1` mutant builds.
  The mutant totals of the fixture are now two shared constants, used by this subtest and the existing assertion.
- F3 (`095ccadd`): `embeddedLineText` gains `a=b: c` and `key:  ` lines, and `TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine` gains one row each, with the later row renumbered.
- F4 (`fde57e7e`): the `mutantBuildRounds` doc names the build from which a refused build drops whole files.

- Gate repair: the comments this round added in `internal/testcover/mutant.go`, `internal/testcover/doc.go`, `internal/testcover/mutantbuild_integration_test.go` and `internal/gatecli/redundant_integration_test.go` were rewritten one sentence per line, none reaching 120 characters.
  No code changed. The tier 0 and tier 1 gate for both packages passes after it.
- Second gate repair: the `embeddedLineText` comment in `internal/testcover/mutantbuild_integration_test.go` still wrapped mid-list, and now breaks at sentence boundaries.
  The tier 0 and tier 1 gate for `internal/testcover` passes after it.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the package doc in `internal/testcover/doc.go` lacked the exemption — fixed as proposed.
- fix-consistency, outside its lens (LOW): the inline total 21 in the new subtest duplicated the sibling assertion's 20 and 1 — fixed as proposed, with shared constants.
- fix-proposal, LOW: the same package-doc staleness — merged with the fix-consistency finding, fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
