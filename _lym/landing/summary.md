# Darn-Write writer runs no gate itself when the cold fixer is on

## What changes

When the Darn-Write row's `verify` gate has a cold fixer, the darn writer and its subagents no longer run `lym gate test`.
The row's `verify` gate tests the committed tree once the writer ends its turn, and the fixer clears its failures before the writer hears of them.
The writer acts only on what the fixer leaves.
With the fixer off, or on the prime where there is no pair slug, the writer's prompt is byte for byte what it was.

`darn-template-write.md`'s Step 4 list is now one `make_change` marker.
`darnengine.DarnTable` fills it with one of two new fragments, read at call time from the stencils directory:

- `darn-fragment-make-change`, today's list, which has the writer test after each commit and its subagents run the gate on their packages;
- `darn-fragment-make-change-gated`, which has the writer and its subagents run no `lym gate test`, drops "each leaving the tree green" from the commit step, and says the writer is re-prompted only with what the fixer leaves.

`DarnInputs` gains `GateFixed`.
`wireDarn` sets it through `writerFixerOn` over the Darn-Write row's `verify` element, so the prompt drops the gate runs exactly when `gateFixerSeam` wraps that gate: a pair slug and a non-empty `darn.gate_fixer`.
This is the same rule the Plan-Write and Discussion-Write prompts already follow for their validators.

## Design decisions

- **The marker covers the whole Step 4 list.**
  The gate runs are named in items 1, 2, 3 and 6, and the ungated text had to stay unchanged, so the only contiguous span holding all of them is the list.
  Items 4 and 5 are therefore repeated in both fragments.
- **`subagent_model` moves into the fragments.**
  Item 2 carries the subagent model, so `DarnTable` fills the chosen fragment's `subagent_model` before it sets `make_change`.
  The writer template no longer declares `subagent_model`; it declares `make_change`.
  An older binary over the newer stencils would leave `make_change` unfilled.
  The peer run `seat-stencil-marker-unfilled` adds a Preflight check that refuses such a skew.

## Tests

- `TestDarnSpec` (`internal/darnengine/spec_test.go`) pins both renderings.
  The ungated prompt carries the gate-run step and the subagent gate rule and lacks the no-gate step.
  The gated prompt carries the no-gate step, the subagent no-gate instruction and the fixer hand-off, and lacks both gate-run lines and "each leaving the tree green".
- `TestStencils_TopLevelMarkers` pins `subagent_model` as each new fragment's only marker.

## To check by hand

- Run a darn task in a pair with `darn.gate_fixer` set and confirm the writer runs no `lym gate test` and that a failing verify gate goes to the fixer first.
