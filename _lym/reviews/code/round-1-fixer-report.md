---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed test-handle row, where card 1 itself creates plan:internal/watch_test#helper, passes with the test-handle branch removed, since a func the card creates is refused anyway (internal/planparser/latercaller.go, refCanHoldProductionCaller). The row has card 1 edit that handle without creating it. Every other proposed row is as proposed."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposal leaves every own func declaration of the later card out of the declared map. That would count a func whose head names a testing type, which is test code, as a production caller. Such a func stays in the map."
  - id: F6
    disposition: fixed
---

## Changes

- F1: `create-caller-later` skips a function declared in `internal/testkit`, `internal/gitkit`, `internal/hubforge` or below one.
  Rows cover an exact directory and a nested one.
  The checks spec and the landing summary state the skip and the package-internal `_test.go` helper limit.
- F2 (fixed differently): rows for the group filter and the handle-ref branches.
  The test-handle row does not follow the proposal, as the frontmatter says.
- F3: rows for the test-name skip, two later callers, a bare symbol and `_test` refs; every row now checks the finding's `Ref`.
- F4: `loom-plan-spec.md` names `create-caller-later` in the Plan vs. schedule section.
- F5 (fixed differently): a later card's own new func counts as able to call, unless it is test code.
  Rows cover the counted and the `testing.` case.
- F6: the landing summary's blast radius names the gates and dispatch validation, and the effect on a run in flight.

Commits: one for F1, F2, F3, F5 and F6, one for F4, one for the self-check fixes, and one for the gate's comment findings.

Gate repair: the round gate refused comment lines in `internal/planparser/latercaller.go` and `cmd/lym/testsupportdirs_linux_test.go` that wrapped mid-sentence.
Those comments are rewritten with one sentence per line; no code changed.
The tier0 run over `internal/planparser` and `cmd/lym` passed again after the rewrite; the tier1 run below predates it.
`_lym/landing/summary.md` is edited and left uncommitted by this round.

Gates: tier0 over `internal/planparser` and `cmd/lym` passed.
Tier1 over `cmd/lym` and `internal/planglyph`, filtered to `TestTestOnlyCode|TestGlyphChain`, passed, with the reachability guard among the tests that ran; some `TestGlyphChain` tests skipped at that tier.

## Self-check

- fix-code-correctness, LOW, the creating card's own test-support func was not counted as a caller: fixed as proposed. It now counts, with a row.
- fix-consistency, LOW, the test-support list is a second untied copy: fixed with own change. `TestSupportDirectories` is exported, and `cmd/lym/testsupportdirs_linux_test.go` holds it equal to the guard's list.
- fix-consistency, LOW, summary and spec called testkit functions test code: fixed. Both texts now say which funcs count for which card.
- fix-proposal, MEDIUM, a later card that creates a second package-internal test helper raises the finding, against the summary's silence claim: fixed with own change. The plan text cannot tell the two kinds of func apart, so the claim is narrowed and the limit stated in the summary and the checks spec.
- fix-proposal, LOW, untied second copy of the list: fixed, merged with the fix-consistency finding.
- fix-affected-set, LOW, "every caller" with a partial list in the blast radius: fixed. The list now reads as examples.
- fix-affected-set, LOW, the same silence claim: fixed, merged with the fix-proposal finding.
- fix-affected-set, LOW, the two test-name branches had no row: fixed with one row.

The self-check fixes themselves had no second fork pass.

## Disputed

None.

## Deferred

None.

## Rulings

None.
