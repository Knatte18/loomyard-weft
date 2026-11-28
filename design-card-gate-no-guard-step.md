# The card gate leaves the guard step to the batch gate

`card-gate-no-guard-step` · Note · enhancement, gate, webster, weaver

## Seen (`docs/research/webster/webster-time-2026-11-08.md`, lever 1)

The guard step was 7,043 of the card gate's 9,117 s (77%), run 787 times for 226 cards, while the batch gate already runs it.
Dropping it from the card gate saved an estimated 6 to 8 min per Webster row; its failures would surface at the batch gate, about 60 s a fix round.

## Idea

Run the guard step only in the batch gate, for Webster and Weaver alike.
Keep or drop this note on the figures the implementer gate-time measurement gives.
