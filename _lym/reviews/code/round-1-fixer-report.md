# Fixer-report, round 1

## Changes

- F1: `internal/websterengine/beginbatch_test.go`.
  The begin-batch prompt test gets a row for batch 2, the last batch, asserting the gate line ends in `--main-packages`.
  The existing batch 1 row now also forbids `--card 1-json-flag --main-packages`.
  The review's premise was partly off: that row's substring already ended in a closing backtick.
  The real gap was the last batch, which no begin-batch row covered.
- F2: `internal/gatecli/detachfront_test.go`.
  `TestTestRequestWords_MainPackages` asserts `words()` renders `--main-packages` once when set and not when unset.
- F3: `contracts/stencils/webster/webster-body-implementer.md` and `internal/websterengine/template_test.go`.
  The `--main-packages` exception is carried into the self-fix rule, the "does not make the report FAILED" sentence and the `status` rule.
  The line 107 phrase now names the rules below.
  The ordered-needle pin in `template_test.go` follows the new `status` sentence.
- F4: `internal/gatecli/scope_test.go` (new) and `internal/gatecli/scopes_integration_test.go`.
  `TestWithMainPackages` covers no mains, adding to a named module, listing once, appending a module the cards leave out, and matching a module dir after cleaning.
  Integration rows cover the `go list` failure refusal (names the flag, the module and the way forward), and `--main-packages` adding nothing under the none and dotnet scopers.
- F5: `internal/impactset/mains_integration_test.go`.
  A subtest lists a `package main` whose `//go:embed` pattern matches no file.
  A scratch check showed `go list` without `-e` exits 1 on it and lists it with `-e`, so the subtest pins `-e`.
  The first fixture, a type error, was replaced after the forks pointed out `go list` does not type-check.
  The two directory error branches stay untested: the finding's fix key names only the `-e` subtest, and `go list` output reaching them was not constructed.

Gate: tier 0 over `internal/gatecli`, `internal/impactset`, `internal/websterengine` and `contracts/stencils` passed.
Tier 1 over `internal/gatecli` and `internal/impactset` passed.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, MEDIUM: the F5 type-error fixture does not pin `-e`. Fixed with my own change: an embed-pattern fixture instead of the forks' unresolvable import, which a scratch `go list` run showed also lists without `-e`.
- fix-proposal, MEDIUM: same finding, merged with the one above.
- fix-proposal, LOW: the none and dotnet rows are weak pins. Fixed as proposed by the review; no change, since they pin the contract the finding named and no stronger assertion exists.
- fix-consistency, LOW: line 131 of the stencil lacked the exception. Fixed as proposed, merged with the fix-proposal and fix-affected-set findings of the same line.
- fix-affected-set, LOW: same line 131 finding, merged.
