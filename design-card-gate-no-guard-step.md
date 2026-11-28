# The card gate leaves the guard step to the batch gate

`card-gate-no-guard-step` · Note · enhancement, gate, webster, weaver

## Seen (`docs/research/webster/webster-time-2026-11-08.md`, lever 1)

The guard step was 7,043 of the card gate's 9,117 s (77%), run 787 times for 226 cards, while the batch gate already runs it.
Dropping it from the card gate saved an estimated 6 to 8 min per Webster row; its failures would surface at the batch gate, about 60 s a fix round.

## Measured again (`docs/research/webster/gate-time-2026-11-28.md`)

The lever still holds with a smaller net saving: 3 to 8 min per row, ranked fifth of the report's levers.
The card guard step is 8.1 of 28.7 card gate minutes per row, and one more batch gate (3.0 min) per failed guard step costs up to 5.5 min of that.
It overlaps the report's lever C (card gate over the card's own packages), which would make the guard step take back about 4.4 min.

## Idea

Run the guard step only in the batch gate, for Webster and Weaver alike.
Keep or drop this note on those figures.
