# loom status --watch prints every poll while a verify runs

`watch-elapsed-churn` · Task · bug, loom, cli · running · Finalize

`lym loom status --watch` prints a line every poll while a verify runs, because the composed activity carries the verify's elapsed seconds.

## Seen

On the `cs` hub (2026-11-03), watching `pipeflow-reynolds-helper` at Darn-Write printed one line per second:

```
loom running | now Darn-Write | last Drift-Await → done | waiting Darn-Write: verify running 0s (attempt 1; lym gate test --wait --gate darn-write)
loom running | now Darn-Write | last Drift-Await → done | waiting Darn-Write: verify running 1s (attempt 1; lym gate test --wait --gate darn-write)
…
loom running | now Darn-Write | last Drift-Await → done | waiting Darn-Write: verify running 6s (attempt 1; lym gate test --wait --gate darn-write)
```

The verb's help promises "a line only when the composed activity actually CHANGES rather than once per poll", but `internal/loomcli/reviewstatus.go` puts `formatElapsed(elapsed)` into the wait text (`<producer>: verify running <elapsed> (<detail>)`, and the gate-slot wait form), so the activity changes on every poll.
An agent or Monitor that watches a run gets one event per second for the length of each gate, which floods its context.

## Expected

The watch's change test ignores the elapsed counter: it prints when the row, the state, the wait's kind or its detail changes, not when only the elapsed time grows.
The one-shot status keeps showing the elapsed time.
