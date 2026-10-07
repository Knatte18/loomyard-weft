---
format: 5
approved: true
---

# Plan: card-batching

Cut the tokens Webster's implementer forks spend by running small, related cards in one fork.
The plan measures first: `tools/tokencount` reports each Webster fork's tokens and the cards it ran, and a calibration report sets the new estimator beside those measurements on past runs.
Then a cost-model batchifier in `internal/batcher` splits the card sequence into contiguous batches with the lowest estimated cost under hard limits, selected through named profiles in `batcher.yaml` with identity staying the default.
Webster records the partition once per run in `state.json`, runs batches in the batchifier's order and re-batches only the unbegun tail at a rebaseline.

## Card Index

1 — tokencount-fork-report — tokencount lists each Webster fork with its cards, messages, peak context and weight
2 — batcher-estimator — per-card estimate, size source, segment-cost function and profile weights parse
3 — calibration-report — estimator on past plans against each run's base tree beside measured fork weight
4 — recorded-partition — Batcher.Batch takes the plan, cards and a size source, batches carry profile and estimate; webster records the partition at first init, every verb reads it, and batches run in the batchifier's order under an order assertion
5 — cost-batchifier — hard limits and an exact DP over contiguous segments; plan spec drops Batchable?
6 — batcher-profiles — named profiles and active: resolution, load errors naming batcher.yaml, template profiles
7 — rebaseline-tail — rebaseline keeps begun batches and re-batches the unbegun tail
8 — calibrate-template — rescale the cautious profile from the calibration report when the fit is good

## Shared Decisions

- **Signature.** `Batcher.Batch(plan *planparser.Plan, cards []planparser.Card, sizes SizeSource) ([]Batch, error)`: Decision 9's cards and size source, plus the plan, which carries the glyph language the estimator needs to map refs to files; `cards` is a contiguous run of `plan.Cards`.
- **Ref to file.** A target or `Uses` entry weighs a file's lines only when `planparser.RefFile` maps it to a file; a glyph or handle whose unit is a package has no file without quarry, so it contributes its message count and its directory's package and tests entries only. Member-level precision waits on `plan-glyph-precision`.
- **Coefficient keys.** A profile's `weights:` keys are `startup_context`, `fork_messages`, `target_messages`, `test_file_messages`, `uses_messages`, `context_per_line`, `package_context`.
- **Past-run lookups.** A run's plan is read from the newest `loom: plan artifacts for <slug>` commit before its earliest Webster fork, in the repository `-history` names; its base tree is the `start_sha` of the earliest `lyx webster begin-batch` envelope in its Webster transcripts, the HEAD before its first batch forked, read from the current repository. A run or card that cannot be reconstructed is listed as skipped with its reason.
- **Batch identity.** A batch is numbered and named by its first card, as `batchIdentity` does today, so `begin-batch NN` names a batch by its first card's number.
- **No commit for a poor fit.** Card 8 may end without a commit when the calibration fit is poor; its report says why.

## verify:

go build ./...
go test ./...
go test -tags integration ./internal/gitrepo/... ./internal/websterengine/... ./internal/webstercli/... ./internal/loomshed/... ./tools/tokencount/...
go test -tags tmux ./internal/webstercli/...
go vet -tags llm ./internal/webstercli/...
