# Batchifier dry run and profile weights that actually batch

`batchifier-dry-run` · Note · enhancement, webster

Found at card-batching's PR-Gate (PR #423), 2026-10-07.

- No lyx verb prints the partition before a run: `lyx webster validate` computes it but reports only valid/cards/scope, and `lyx webster status` lists batches only once `state.json` exists. Add `lyx webster validate --batches` (or a tool) that prints each batch with its estimate for a plan and a named profile, so a profile is chosen on numbers, not guesses.
- By hand with `estimate.go` and the template weights, a small card costs about 0.87M, so `budget` 2M fits two small cards and a card with 2+ targets and ~15 test files (~1.8M) exceeds `alone_above` 1.2M and runs alone. `cautious` therefore batches at most about two small cards; raising `max_cards` alone changes nothing, `budget` and `alone_above` must rise with it.
- `tools/tokencount` has both `-weft` (finished runs) and `-history` (calibration) naming the same repository; one default would do.