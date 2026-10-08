# Loom goto back to Plan after Webster started leaves webster state stale; Webster hard-fails on plan fingerprint mismatch

`goto-plan-stale-webster` · Note · bug, loom, webster

Imported from [issue #466](https://github.com/Knatte18/loomyard/issues/466).

## What happened

A loom run completed the Plan, Batchifier and Webster rows, then the Publish row went stuck and the run routed back (`goto`) to `Plan-Bouncer` several times.
The Plan loop edited the plan, and `Batchifier` re-ran and finished (`done`).
The next `Webster` step then halted the run as `failed` with anomaly `producer-hard-failure`:

```
webster: on-disk plan fingerprint does not match this run's recorded state: on-disk plan fingerprint e5774b7e... does not match this run's recorded fingerprint c14834a3...; the plan changed since state.json was created; way forward: 00-overview.md changed and is never rebaselined; restore it with "lyx webster restore-plan", or 1) lyx webster reset --to start; 2) re-step the Webster row
```

The run had 49 history entries when it halted.

## Why this is abnormal

The loom's own routing sent the run back through Plan after Webster had already created `state.json`, and then re-ran Webster against that stale state.
The recorded fingerprint can only match if the plan is untouched, so a legitimate Plan-loop edit made the next Webster step fail by construction.
Neither the `goto` to Plan nor the re-run `Batchifier` row reset or rebaselined webster state, so the failure was foreseeable from the recipe's own wiring rather than an operator or task-code error.
The error text offers manual recovery (`lyx webster restore-plan`, or `lyx webster reset --to start` and re-step), which a driver could have applied itself.

## What to do

When the loom routes back to a row upstream of Webster after Webster state exists, reset webster state (equivalent to `lyx webster reset --to start`) as part of that route, or have the Webster row detect a changed plan produced by an upstream re-run and rebaseline instead of failing.
Alternatively, make the recipe validate at the `goto` that a Plan re-entry invalidates downstream webster state.

Provenance: task slug `shuttle-send`; notes: `loom-halt.md`.
