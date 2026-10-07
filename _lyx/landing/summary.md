# Measured batch start context; plan gate refuses fabric-repo commands

The cost batchifier now prices a fork's start from Merriam's growing context instead of a constant, and `tools/tokencount -calibrate` fits and reports the numbers that model uses.
The plan gate also stops a class of plan that the implementer audit later failed with no cheap way out, and `accept-audit --batch` gains a route for the harmless case.

Closes #424
Closes #432

## Start-context model (`internal/batcher`)

- `startup_context` is replaced by `master_base` (Merriam's context at its first fork) and `batch_growth` (context gained per finished batch).
  The batch at position k starts at `master_base + (k−1) × batch_growth`.
  A profile still carrying `startup_context` fails to load with an error naming `batcher.yaml`, the profile, the retired key and its replacements.
- The dynamic program now runs over (batches so far, last card placed).
  Its objective is unchanged: fewest batches, then lowest largest peak, then shortest last batch.
- `budget` still limits only segments of two or more cards.
  The start grows without a cap, so on a long plan a card can exceed `budget` alone and run as a one-card batch.
  Nothing halts or warns; the recorded `Estimate` and position show it, and the operator lowers `max_cards` or raises `budget`.
- The batchifier is told how many batches run before its first card: 0 for first init and `validate`, the kept-batch count for a rebaseline.
  The count assumes Merriam was never resumed, so after a resume the estimate is conservative.
  It is too low when Merriam does unmodelled work between batches, and only the budget's headroom absorbs that.
- Each batch's `Breakdown` records `master_base`, `batch_growth` and the position.
  State recorded with `startup_context` still loads.
- `master_base` is a fitted constant, not a sum over Merriam's files: what Merriam reads before its first fork is the agent's choice, and the measured spread is about 6%.
- The template's `cautious` profile takes `master_base: 52000` and `batch_growth: 7000` from three finished `cautious` runs (twelve forks).
  The in-fork coefficients were fitted under the old constant and await a re-fit from the new report.

## `tools/tokencount`

- One flag names the fabric repo for tags, plan commits and webster records; `-history` is removed and the default is the current worktree's sibling, so `-calibrate <profile>` needs nothing else.
- Calibration reports each fork's measured start, position and estimated start, and fits the two coefficients by least squares over position − 1.
  Position restarts per Merriam session.
  With fewer than two distinct positions the fit is marked not fitted, and a negative coefficient is marked unusable.
  The fit is a report; nothing writes `batcher.yaml`.
- Every fork, one card or many, gets a peak estimate at its position, plus an in-fork growth ratio per run and overall.
- A per-batch cost table and a per-profile summary (weight per card, failed batches, recoveries, Webster-Review findings per card) are new.
  Runs without a recorded partition count as `identity`; a run restarted with `--fresh`, or whose state or review files cannot be read, is listed with the reason and left out of the profile lines.
  Findings are read only through `burlerengine`'s `ParseReview`.

## Plans run in card order

`loom-plan-spec.md` now states that a valid plan is executable strictly in card-number order and Webster never reorders it.
The plan template and the plan-review rubric point at that section.
This was already enforced by Plan-Gate checks and `CheckBatchOrder`, but the spec still said the opposite.

## Fabric-repo commands in plans

- New Plan-Gate check `card-fabric-reference` refuses a card that orders a command naming the fabric repo.
  It scans a card's `**Verify:**` value, the `## verify:` section, and the fenced blocks and inline code spans of every plan file.
  A fabric-repo sibling path is flagged in all of those; a fabric-driving spelling only in the Verify value, the verify section and fenced blocks, since a span documenting a command opens with it.
  It runs inside the plan-format validation that both gate sites and `validate-plan` share.
- The check and the implementer audit share one location-free matching rule, declared on the fabric side (`internal/fabricengine/refscanner.go`) and handed to `planparser` through a seam, so `planparser` imports neither the fabric side nor the suffix package.
  The audit's own matches are unchanged, and it keeps its worktree-specific exact-path branch.
- `accept-audit --batch NN` now also accepts a failed batch that has commits when every uncheckable entry is a pathless fabric-reference finding whose recorded command is read-only.
  Read-only means built only from `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq` and `cut`, joined by pipes, `;` or `&&`, with no redirection, substitution or backgrounding.
  `sort` and `uniq` are excluded because they can write.
  One entry that is not read-only, or that has no recorded command, refuses the whole call with the existing reset steps.
  The implementer audit still fails such a batch; only the explicit accept route widens.
- The plan template steers corpus and fixture cards to committed test data and names `card-fabric-reference` by ID.
- `refusal-spec.md` and the `loom-plan-spec.md` check row change in this commit.

## Check by hand

- The new gate misses a spelling inside an inline span or prose, `bash -c "…"` or a variable, and refuses a path documented in an inline span or a fenced block carrying file content; the implementer audit remains the guard for the rest.
- The `cautious` coefficients come from a small sample of 3–5 batch runs; a long plan's late growth is extrapolated.
  Run `go run ./tools/tokencount -calibrate cautious` against finished runs after landing to check the fit; it was not run here.
- `cmd/lyx/testdata/test-budget.yaml` rows rose for `tools/tokencount`, `internal/fabricengine`, `internal/planparser` and `internal/websterengine` because the change adds tests.
