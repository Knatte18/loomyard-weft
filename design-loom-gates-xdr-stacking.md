# Cortex XDR holds stack up across a loom run's card, batch, round and whole-suite gates

`loom-gates-xdr-stacking` · Note · enhancement, webster, loom, gate

## Seen (orch, 2026-10-20)

On OSL-1046, Cortex XDR holds every test binary with an unseen hash for 8-10 s before it runs (`docs/benchmarks/gate-test-xdr.md`).
A loom run pays that hold at every gate that tests a package the run just changed, and those gates stack:

- the card gate after every card, over the card's own packages (`pattern/PATTERN-test-speed.md`, Gate per tier); each card changes a package, so each card gate links at least one new binary and waits out its hold, and a single-package test gets no overlap from parallelism;
- the batch gate once per batch, `--tags integration` over the batch's packages;
- Webster's verify gate and Publish, each the plan's whole verify, 3-6 min on this machine depending on how widely imported the changed packages are, plus `publish_verify` (the tmux tier) at Publish;
- every Webster-Burler-Fix and Merge-Burler-Fix round gate, over the impacted set.

Finalize and Merge-In skip their verify when the tree is the one already verified (`pattern/PATTERN-verified-tree.md`), so the stacking is in the card, batch, round and whole-suite gates, not in repeats on an unchanged tree.
For a plan of 10-15 cards the card gates alone wait out 10-15 holds, roughly 2-3 min of nothing but XDR, before any test time.
Webster's card, batch and round gates are already scoped; only its plan-level verify gate runs the whole plan verify (`internal/loomshed/doc.go`), and Publish runs it again, which hits the cache when the parent has not moved.
So the stacking is mostly the many small gates that each link a new binary, not an unscoped Webster.

## Ideas (orch, not decided)

- Measure first: clock one ordinary loom run on OSL-1046 per gate kind, so the split between XDR holds and test time is known.
- Dropped (2026-10-20): building test binaries in the background while the next card is written.
  The build would read a working tree the next card is editing, so it can compile a half-written tree whose hashes no later gate uses, and a later card that changes a dependency changes the hashes anyway; building from the card's commit in a separate checkout costs more than it saves.

## Operator decision (2026-10-20)

Webster's plan-level verify gate keeps the whole suite: Webster implements loom's large tasks, which touch much, so it checks everything before it reports done.
Scoping that gate is not an option here; darn, for small changes, is where gates scope to the diff.

The card gate keeps its tests too: a card is atomic by design, it leaves every test passing, so a failure points at exactly the card that caused it.
Moving the tests to the batch gate would lose that, so it is not an option either.
What remains is measuring the hold's share of a few ordinary loom runs on OSL-1046 before anything is built.
