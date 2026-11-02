# insert-card renumbers only <N>-<slug> ids, leaving bare 'card N' mentions stale

`insert-card-bare-mentions` · Note · bug, loom

Imported from [issue #5](https://github.com/Knatte18/loomyard/issues/5).

## What happened

Inserting a card into a plan renumbered every following card, but `insert-card` rewrote only the `<N>-<slug>` ids.
Bare "card N" mentions in plan prose (about twenty) stayed at the old numbers and had to be renumbered by hand.

## Why it is abnormal

The renumber is partial, so the plan is left internally inconsistent: references by number now point at the wrong cards, with no warning from the verb.

## What to do

Have `insert-card` also rewrite bare "card N" references, or report each one it leaves unrewritten so the planner can fix them.

Provenance: task `warp-visibility`, note `burler-plan-r1.md`.



_Moved from #910 in the repository before it was recreated._
