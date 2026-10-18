# Plan cards mark additive edits so shared spots stop chaining the card DAG

`plan-extends-label` · Note · enhancement, loom, webster, burler

## What was seen

`tools/plandag` (landed `c03af73d0`) was run on nine recorded plans of 12 to 20 cards (2026-10-18, plan artifacts from the records repo).
The best parallel speedup, cards over critical path with every card counted equal, came out at about 1.8x median:

| Plan | Cards | Critical path | Best speedup |
|---|---|---|---|
| `overview-seats` | 20 | 11 | 1.8x |
| `webster-fork-stencil-fixes` | 17 | 9 | 1.9x |
| `batten-start-speed` | 16 | 9 | 1.8x |
| `claimed-scope-edit` | 14 | 8 | 1.75x |
| `run-pane-timing` | 14 | 5 | 2.8x |
| `engine-registry` | 14 | 11 | 1.3x |
| `engine-model-choice` | 14 | 8 | 1.75x |
| `shed-tackler` | 14 | 7 | 2.0x |
| `report-timing` | 12 | 9 | 1.3x |

Much of each chain comes from a few shared spots that many cards change by adding to them, not by changing what is there:

- docs: the package `doc.go` (4 to 7 cards per plan), `docs/overview.md` (3 to 5), `PATTERN.md`;
- specs: `contracts/specs/refusal-spec.md` (4 to 5 cards);
- CLI wiring: `loomCLI.wire`, `battenCLI.wire`, `fabriccli#addWeftVerbs` (3 to 6 cards), where each card adds a flag or a verb;
- `cmd/lyx/testdata/test-budget.yaml` (7 to 9 cards in three plans), since removed by `test-budget-remove`.

Real overlaps also exist, such as six cards changing `websterengine#Run` in `engine-model-choice`; those stay dependencies.

## Idea (agreed with the operator 2026-10-18)

A plan card states how it changes a shared spot, not only that it changes it:

1. A new card label, working name `Extends`, beside the change labels and `Uses`: the card only adds to the ref (a flag, a list line, a doc paragraph, a spec entry) and changes nothing already there.
2. Two `Extends` on the same ref are no dependency in the card DAG; `Extends` against a real change still is.
3. Git merges additions in different hunks by itself; two additions on the same lines are a simple conflict for the merge-back resolver of parallel batches (see note `parallel-batch-parts`).
4. Plan review checks the label: an `Extends` on a ref the card in fact rewrites is a finding, like a missing `Uses` today.
5. Docs use `Extends` too, so the card that writes the code also writes its docs, and no separate final docs card is needed.

The risk is a wrongly labelled `Extends`, which surfaces as a harder merge conflict at merge-back, never silently.

## First step: measure before building

Before the plan format changes, extend `tools/plandag` (darn, review 0) to recompute the DAG with additive refs guessed by path and name (doc files, `docs/overview.md`, `PATTERN.md`, `contracts/specs/*`, CLI wire functions), treating change-against-change on them as no dependency.
Rerun it on the plans above and compare critical paths; the size of the drop decides whether the label is worth building.

### Rough estimate (orch, 2026-10-18)

A throwaway awk pass over the plandag matrices recomputed each critical path with plandag's own overlap rule; with nothing skipped it reproduced every critical path above.
Columns skipped for change-against-change only: `test-budget.yaml` (already gone), then doc files, `docs/`, `PATTERN.md` and `contracts/specs/`, then also wire functions guessed by name (`wire*`, `addWeftVerbs`).
Use against change still counts.

| Plan | Cards | Today (no test-budget) | Docs and specs additive | Plus wire additive |
|---|---|---|---|---|
| `batten-start-speed` | 16 | 9 | 8 | 8 |
| `claimed-scope-edit` | 14 | 8 | 8 | 6 |
| `engine-model-choice` | 14 | 8 | 7 | 6 |
| `engine-registry` | 14 | 7 | 5 | 5 |
| `overview-seats` | 20 | 11 | 9 | 8 |
| `report-timing` | 12 | 4 | 3 | 3 |
| `run-pane-timing` | 14 | 5 | 5 | 5 |
| `shed-tackler` | 14 | 5 | 5 | 4 |
| `webster-fork-stencil-fixes` | 17 | 8 | 8 | 8 |

Median best speedup, cards over critical path with every card counted equal: about 2.0x today, about 2.5x with docs, specs and wire functions additive.
Removing `test-budget.yaml` already lifted the median from about 1.8x.
Docs and specs alone give the smaller part; wire functions give about as much again.

## Side finding

`fabriccli#addWeftVerbs` holds shared hidden flags and pre-run logic beside the verb list, which is why so many cards touch it; a thin wire function, only the list, with flags and logic beside each command, would remove much of that hot spot.
Orch view, not a decision: this is a separate code cleanup, to become its own note if the measurement shows wire functions matter.
