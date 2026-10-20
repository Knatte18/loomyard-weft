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
- Hide the hold by overlap: after each card's commit, build in the background the test binaries that card's change made new and no later card changes again, such as reverse dependencies and guard tests, so XDR has scanned them before the batch gate or the whole suite needs them; eight fresh binaries started at once finished in about the time of one.
  A card gate's own binary cannot be built ahead, since its hash changes only once the card is written; the gain is uncertain and needs a measurement first.
- Fewer gates that link new binaries: let the card gate on such a machine run only the lints, and leave the tests to the batch gate.

## Operator decision (2026-10-20)

Webster's plan-level verify gate keeps the whole suite: Webster implements loom's large tasks, which touch much, so it checks everything before it reports done.
Scoping that gate is not an option here; darn, for small changes, is where gates scope to the diff.
