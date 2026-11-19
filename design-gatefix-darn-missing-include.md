# gatefixer: gatefix-body-darn prompt fails on missing include loom-fragment-loosened-check

`gatefix-darn-missing-include` · Task · bug, loom, stencil · running · Drift-Recheck

Imported from [issue #149](https://github.com/Knatte18/loomyard/issues/149).

## What happened

The loom run halted `failed` at `Darn-Write` with anomaly `producer-hard-failure`.
The error chain ended in the gate fixer's prompt build:

```
shedadapters: Darn-Write (seats): seat run: seatengine: chair wait: shuttle: gate: gatefixer: build the fixer's launch: gatefixer: fill the gatefix-body-darn prompt: stencil: include "loom-fragment-loosened-check" has no entry in the includes
```

## Why it is abnormal

The `gatefix-body-darn` prompt references the include `loom-fragment-loosened-check`, but the gate fixer's launch supplies no entry for it.
The failure is deterministic prompt wiring, not a transient fault, and it kills the whole run at the point where the gate fixer is first needed.

## What to do

Add the `loom-fragment-loosened-check` entry to the includes the gatefixer passes when it fills `gatefix-body-darn`, or drop the include from the prompt.
Add a test that fills every gatefix body prompt with the production includes map, so a missing include fails in CI rather than mid-run.

Provenance: task `redundant-check-fixes`; note `loom-halt-2.md`.
