---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## What changed

All changes are in `internal/gatecli/redfirstdotnet_integration_test.go`, in commits `bdffca7c5` and `9bfd8d596`.

- **F1:** fixed by the second branch of the proposed fix: the green row stays dropped.
  I added it back as a third parallel row, measured it, then removed it, because it lengthens the test.
  Commit `bdffca7c5` carries a `Loosened-Check:` trailer naming the dropped exit and envelope-text assertions, with the figures.
  What remains uncovered: no real `dotnet` run of red-first exits `gateslot.RedFirstGreenExit`.
- **F2:** the red row asserts the `tests` list (`bdffca7c5`) and the tier run's counts, `discovered 2 passed 1 failed 1` (`9bfd8d596`).
  The counts show that the passing test ran and passed.
- **F3:** the unmeasured share-of-time clause is gone (`bdffca7c5`).
  The comment says each row is slow, so the sentences that follow keep their reason (`9bfd8d596`).

### F1 measurement

Alternating two-row and three-row runs, each with `TestGateTest_DotnetFixture` beside it in `internal/gatecli`.
A figure is the longest parallel row of that run.

| pair | two rows | load at start | three rows | load at start |
|---|---|---|---|---|
| 1 | 14.16 s | 34.32 | 19.37 s | 35.71 |
| 2 | 12.51 s | 46.82 | 18.12 s | 44.77 |
| 3 | 14.11 s | 51.56 | 23.61 s | 49.54 |

The trailer in `bdffca7c5` says the load was 28 to 52.
The right range is 34 to 52; the body of `9bfd8d596` says so, and history is not rewritten.

### Verification

On the final tree, `TestGateRedFirst_Dotnet` (both rows ran) and `TestGateTest_DotnetFixture` passed through tier1, which builds and vets first.
The tier0 run of `internal/testkit`, which holds the formatting guard, passed.
The full diff-scoped tiers were not run.

## Self-check

- fix-code-correctness, LOW, the `tests` needle shows selection, not a run: fixed differently from the lens's suggestion to only record the limit; the row asserts the run's counts (`9bfd8d596`).
- fix-code-correctness, LOW, wrong load range in the trailer: fixed as above, in the body of `9bfd8d596` and in this report.
- fix-consistency, LOW, wrong load range in the trailer: duplicate of the line above, fixed.
- fix-consistency, LOW, comment lost the reason for "So" and "For the same reason": fixed (`9bfd8d596`).
- fix-consistency, LOW, `_lym/landing/summary.md` names only the dropped exit code and gives no three-row figure: deferred, see below.
- fix-proposal, LOW, the `tests` needle shows selection, not a run: fixed (`9bfd8d596`).
- fix-proposal, LOW, comment lost its reason: duplicate, fixed.
- fix-affected-set: no findings; its note that the needle shows selection only is the duplicate above.

## Disputed

## Deferred

- Self-check item on `_lym/landing/summary.md`: the file is a landing record outside this round's write surface.
  Its text is still true but incomplete: the dropped row also asserted "does not show the bug" in the envelope, and the figures for keeping the drop are in the table above.

## Rulings
