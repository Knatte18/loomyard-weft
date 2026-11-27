# Remove the coverage maps and every user of them

Removes the stored per-test coverage maps and everything that read them.
`docs/research/testmap-worth-evaluation.md` recommends this:
the tools worked, but they cost far more than they gave.

## What goes

- The `reach`, `redundancy` and `mutation` report steps of `gate test`, and the `reports` key of a gate key.
- The `gate reach`, `gate mutate` and `gate redundant` commands, and the `--map` flag of `gate test`.
- The reach turn of the batch gate and the `verify` gate, which sent the writer a note of unreached functions.
- The three report sections of the Code-Review rubric and their stencil markers.
- The `redundancy_*`, `reach_budget_min` and `mutation_*` keys of `gate.yaml`.
- The map and mutation code of `internal/testcover` and `internal/testreach`, with the tests of everything removed.
- `PATTERN-coverage-map-reader` and its guard test, and the map text of the other docs.

## What stays

- `cmd/testtiming -redundancy`, for an occasional sweep, with the part of `internal/testcover` it needs.
- `gate red-first`.
  `internal/testreach` now only finds the tests a diff adds or changes.
  The helpers red-first shared with the removed steps are now in `internal/gatecli/gomodule.go` and `redfirst.go`.

## Design notes

- A `gate.yaml` that still holds `reports` or the other removed keys loads, and they are ignored.
  The retired `coverage` scope is still refused.
- Code that only served the report steps running beside a tier is gone too.
  The memory guard no longer leaves any test binary to another guard,
  and the process cap counts every marked process without checking its ancestry.
- `shuttleengine.RerunMark` is removed with the reach turn, its only reader.
- `proc.MemoryCapCoversTree` now has only test callers, and is listed as allowed in the test-only-code guard.

## By hand after landing

- Run the CLI's `config reconcile --layer default --apply` from the hub's prime worktree.
  It drops the removed top-level keys of `gate.yaml`.
  Without `--apply` it is a dry run.
  The automatic reconcile after a binary change does not remove them.
- That reconcile keeps the `reports: []` entries of this repository's own hub `gate.yaml`,
  since it does not prune inside a `gates` entry.
  They are ignored at load.
  To remove one, run `config gate --unset gates.<name>.reports` from the prime worktree;
  this is read from the code and has not been run.
- Maps already on disk are not deleted.
  The `testcover` directories in the state directories of the board checkout and the worktrees can be removed.

## Merge danger

**Door:** two-way
**Blast radius:** gate
