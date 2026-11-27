# Remove the coverage maps and every user of them, keeping the redundancy sweep and red-first

`remove-testmap` · Task · enhancement, testmap, gate, darn, loom, stencil, cleanup · draft

Depends on: [`memory-guard-infra-stop`](design-memory-guard-infra-stop.md), [`cli-form-test-in-card-gate`](design-cli-form-test-in-card-gate.md)

## Decision (operator, 2026-11-27)

The evaluation in `docs/research/testmap-worth-evaluation.md` (landed `8989232ea`) recommends removing the stored coverage maps and every user of them, and the operator chose that.
The tools work, but few tests turned out redundant, and they cost far more than they gave:
none of the 88 code-review findings that used a report found a production defect on its own,
and the report steps took 7.0 hours of work and 2.8 hours of slot waiting over the window measured.
The maps and their users are about 11,400 non-test and 10,300 test lines.

## Remove

- The `reach`, `redundancy` and `mutation` report steps of `lym gate test`, the `reports` key of a gate key, and their stencil text: the reach turn, the review markers and every prompt or rubric line that names a report.
- `lym gate reach`, `lym gate mutate`, `lym gate redundant` and `lym gate test --map`, with their help, docs and PATTERN text.
- The stored maps, their writers and readers, their upkeep, and any dobby or row step that starts a map run.
- The tests of everything removed.

## Keep

- `cmd/testtiming -redundancy`, for an occasional sweep, and the part of `internal/testcover` it needs.
- The red-first gate and `internal/testreach`, which use no map.

## Hub config

The loomyard hub's `gate.yaml` sets `reports: []` on several gate keys.
When the key leaves the template, the change description names the `lym config reconcile --layer default --apply` step the hub needs.
