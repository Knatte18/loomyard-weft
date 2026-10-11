# Seats chair waiting on its advisor raises held notices to the orch

`chair-wait-held-notice` · Note · enhancement, shuttle, loom

## Seen (2026-10-11, chair-blind bench pair twb-seat-lead-1)

A seats chair that sends a batch to its advisor ends its turn to wait for the answer.
Shuttle reads a turn end without output files as held and sends the orch a shuttle notice each time, for example `Batch 1 sent to the advisor; waiting for its answer.`
One Discussion raised three such notices in a few minutes; each wakes the orch, which re-reads its context to do nothing.

## Wanted

- A chair with an advisor question outstanding is not held: shuttle knows the chair is waiting on a seat and stays quiet, or the chair waits inside its turn (for example with `lyx shuttle await` on the advisor's answer) instead of ending it.
- Related: note `fork-wait-noise` covers the same pattern for webster forks.