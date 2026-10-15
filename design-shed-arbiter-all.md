# The arbiter in shed for every shed run, batten included

`shed-arbiter-all` · Note · enhancement, shed, batten, loom, config

## Seen (2026-10-15, after `shed-tackler` landed)

The Go driver in `internal/sheddriver` has an optional `Arbiter` seam.
Loom and darn fill it (`internal/loomcli/drivewiring.go`, `ArbiterSeam`), so their stop chain is: rule table, then the tackler (`haiku`), then the arbiter (`opus[high]`), then the orch.
Batten's own lifecycle run fills only the tackler (`internal/battencli/drive.go`, `sheddriver.Deps` has no `Arbiter`), so a stop the tackler escalates goes straight to the orch.
The arbiter's keys (`arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns`) sit in `loom.yaml`, while the tackler's defaults moved to `shed.yaml`.

## Operator view, not a decision (2026-10-15)

The stop chain should be the same for every shed run: if loom and darn have an arbiter, batten should have one too.
The arbiter is there to take load off the orch.

This matches the operator's 2026-10-14 principle that anything every FSM needs belongs in shed, with shed defaults a module's config overrides.

## Idea

- Move the arbiter into shed beside the tackler: its seam wired for every shed-driven recipe, batten included, and its keys in `shed.yaml` with per-module overrides, as `driver_kind` and the tackler keys already are.
- Give batten an arbiter stencil or a recipe line that describes batten's rows (Worktree-Create, Run-Shed, Land, Worktree-Teardown and the rest), so the arbiter can judge a batten stop.
