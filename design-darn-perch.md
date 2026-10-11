# Darn gets a perch review with a configurable budget

`darn-perch` · Task · enhancement, loom, burler, shed · awaiting · PR-Gate

## Wanted (operator, 2026-10-11)

Darn keeps its one writer: the `Darn` row (`DarnWrite`) still reads the board entry, edits, tests and commits everything itself, as today.
After it, a perch reviews the diff as a safety net before Publish: a Bouncer row whose `on_stuck` points at a Burler-round row, whose own `on_stuck` points back, wired the way the loom recipe wires its review segments (`contracts/recipes/loom-recipe.yaml`, e.g. `Webster-Bouncer` and `Webster-Burler`).

## Budget

The perch's round budget is set in darn's config (`darn.yaml`, beside the verify budget), so it is easy to change per hub.
A budget of `0` is valid and means no review: the run goes from `Darn` straight to `Publish`, as today.
State the chosen default and the key's name in the PR description.

## Notes

- `darn-recipe.yaml` row names are durable on-disk identities pinned against `internal/loomshed` `Name*` constants; adding rows needs matching constants, and an in-flight darn run parked past `Darn` must still resume.
- PR-Gate keeps bouncing to `Darn` on a rejection.
- Update the darn recipe's header comment, the darn and loom docs (`doc.go`) and `docs/overview.md` if the execution stack description changes.
