# Round 2 fixer report — plan

Both findings of `round-2-review.md` are fixed.
The `approved:` key in `00-overview.md` is untouched.

## Changed

- **F1** (`11-tmux-tier-green.md`, `00-overview.md`, `15-after-state.md`):
  card 11 now cites the benchmark's failure as recorded at `04d9b57c0` on the pre-fold top-level test, states that it did not reproduce on this tree (the subtest, the `TestSmokeDotFill` scenario and the full `go test -tags tmux ./...` all passed on the recorded machine class), and drops the claim that the harness's `split-window` is the failing call, since that call's error carries no tmux text.
  The card ends in one of two outcomes: the tier passes and the card changes nothing, or a failure is captured first and fixed in `smoke_dotfill_test.go`, with a cause outside that file reported rather than widened into.
  Its `ImpactSummary` follows.
  The overview's card-index line for card 11 and card 15's mention of the test name now match the card and the current test shape.
- **F2** (`09-mark-guards.md`): the six listed files whose only scan reads their own package directory (`battenrecipe`, `battenshed`, `loomrecipe`, `orchengine`, `shedbuild`, `shedrecipe` seam tests) leave the `**Edit:**` list, so the list follows the card's own marking criterion.

## Prior rounds

Round 1's findings F1 to F7 hold: the coexistence rule of card 9 and card 8 is intact, and the other six fixes are untouched.
Round 1 judged card 9's over-inclusion harmless; F2 above trims it anyway, at no cost to coverage, since the criterion leaves those files unmarked.

## Deferred

Nothing deferred.
