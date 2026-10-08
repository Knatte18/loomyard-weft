# Config reconciles itself on the first lyx command after a binary change

`config-auto-reconcile` · Note · enhancement, config

## Why

From webster-dead-ends PR-Gate (2026-10-08): that task retired the batcher keys `master_base` and `startup_context`, and Loom-Preflight now refuses a `batcher.yaml` that still carries them, pointing to `lyx config reconcile --apply`.
After the deploy the hub ran a manual reconcile in the prime and in every in-flight pair.

## Proposal (operator, 2026-10-08)

The trigger is a changed `lyx` binary, not a deploy: `update-plugins.sh` is loomyard's own route, and every other repo only gets a new binary.

- Each hub keeps a stamp under its `.lyx` naming the binary that last reconciled its config.
- The first `lyx` command in the hub that runs under a different binary reconciles the prime and every pair of the hub once, logs what it migrated, and writes the stamp.
- It runs before the command itself, so the orch, `lyx orch start` and `lyx batten run` already work with the new config; it cannot wait for the recipe's `Preflight` row, since the orch runs outside any recipe.
- The recipe's `Preflight` row checks the same stamp as a safety net and runs the reconcile when it is stale.
- A reconcile that cannot apply (a hand-edited conflict) stops the command with the reconcile's own way forward rather than running on a stale config.

## Open

- Concurrency: two `lyx` commands starting at once after a binary change; the reconcile takes a hub lock.
- Whether the same stamp also covers `stencil-drift-behind-main`: a pair behind the new binary is expected after a binary change, not a source-ahead fault.

## Also from that PR-Gate

`merriamFixedContext` is a constant from one measurement, so the system-prompt-and-tools figure drifts silently as Claude Code changes; a calibrate step in `tools/tokencount` could re-measure it.
