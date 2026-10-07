# Cost-model batchifier with named profiles in batcher.yaml

Webster's implementer forks are the largest token cost of a run, because each fork reads a large context afresh.
This change adds a batchifier that runs small, related cards in one fork, and the means to calibrate it before it is switched on.
Nothing groups cards by default: `identity` stays the default profile, and the new `cautious` cost profile ships inactive.

## What changes

- `internal/batcher` gets a card-size estimator and a cost-model batchifier.
  Per card, the estimator reads its targets, their kind (create, edit, delete, rename), its `Uses` entries and the test files of its target directories.
  The sizes come from a size source: the live one is rooted at the told worktree root, and the calibration one reads a git tree.
  A fork's cost is the sum of the context each of its messages reads, so cards sharing files cost less merged, and a card that adds much context costs more merged.
  The per-fork messages are paid once per batch, which is the only term by which merging saves.
- Batches are contiguous segments of the plan's card order, chosen by an exact dynamic program for the lowest total cost.
  Hard limits hold whatever the model says: a card above `alone_above` runs alone, and a batch of two or more cards stays within `max_cards` and `budget`.
  A card over `budget` on its own runs alone, as today.
- `batcher.yaml` takes `active:` naming a profile under `profiles:`, each with a `batchifier` kind (`identity` or `cost`) and, for `cost`, `alone_above`, `budget`, `max_cards` and `weights`.
  An empty `active:` or `identity` with no `profiles:` block resolves to identity, so existing configs keep working.
  An unknown profile or kind, a missing or negative coefficient, a non-positive threshold or `max_cards` below 2 fails to load, and the Batchifier row reports Stuck naming `batcher.yaml`.
- The `Batcher` interface now takes the size source; identity ignores it.
- Webster records the partition once per run in `state.json`, with each batch's profile name and estimate, and every verb reads it from there.
  A file the batches change can no longer regroup cards mid-run.
  A state written before this change recomputes with the identity batchifier, so an in-flight run survives the deploy.
- A rebaseline keeps every begun batch and re-batches only the unbegun tail with the then-active profile.
  A rebaseline whose profile does not load refuses and leaves state unchanged.
- Webster runs batches in the batchifier's output order.
  `SequenceBatches` stops reordering and becomes an assertion: a batch depending on a later batch, or a cycle, is a refusal with a way forward.
  Cycle condensation and the informational `Cycle` warning go.
- `tools/tokencount` reports each Webster fork with its run, the card ids it ran (from the card file names in the fork's prompt), message count, peak context and weight; a fork naming no card is listed as unattributed.
- `tools/tokencount` also gets a calibration report.
  It reads past runs' plans and base trees read-only from git, runs the same segment-cost function as the live batchifier, and writes per card the estimate, the measured weight and their ratio, with a per-run and overall median ratio and spread.
  Runs or cards it cannot reconstruct are listed as skipped with the reason.
  `internal/gitrepo`'s local blob reads gain what this needs.
- The plan spec's `Batchable?` column is retired in favour of the batchifier's rule, since no code enforced it and it said Create and Edit cards never share a fork.
  Package docs, `docs/overview.md`, the refusal spec, the webster spec and the `PATTERN-batcher-registry` entry are updated.

## Design decisions

- The estimator and the calibration report call one segment-cost function, so calibration and live sizing cannot drift apart.
- Ties in the split go to more, smaller batches.
- Reordering cards to bring related ones together is out of scope; the batchifier only cuts the given order.
- Switching the default profile away from identity is left to the operator after calibration.
- The template's `cautious` profile ships with initial coefficients and thresholds, in context tokens.
  The per-fork messages cannot be fitted from past one-card forks, so the saving from merging is only as good as their initial value until real runs under a cost profile are measured.

## To check by hand

- Run the calibration report on the operator's transcripts: `go run ./tools/tokencount -last 40 -calibrate cautious -history <weft repo>`.
  The template's `cautious` coefficients were meant to be rescaled by the median ratio if the fit is good (spread of per-card ratios at most 3).
  The run record shows the calibration batch ended without a commit, and I did not confirm whether the fit was poor or the report could not be produced; check that the template values are the ones you want before switching `active:`.
- That batch's fork ran the calibration command against the fabric (weft) repo, which an implementer fork must not touch; the audit flagged it three times and the batch left a clean tree.
  Check that the weft repo is unchanged.
- Switch `active:` to `cautious` on a scratch run and compare each batch's recorded estimate with its real token use.
- Merged batches localise failures less precisely, and `recover-batch` restarts the whole batch.
