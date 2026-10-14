# Twill: spawn lens forks after the files are written

`twill-spawn-after` · Task · enhancement, loom, stencil · running · Darn-Burler-Fix

## Decided (operator, 2026-10-13)

Twill's writer spawns its lens forks after it has written the reviewed files and its self-check has passed, not before writing.
This holds for both twill hosts, Discussion-Write and Plan-Write.
The benches behind it are in `twill-fork-timing` and `docs/research/twill-fork-timing-bench.md` (Discussion: $4.44 against $5.43 per run, quality held) and `docs/research/twill-fork-timing-plan-bench.md` (Plan: lens forks $0.94 against $1.60 per run).

## Scope

- `contracts/stencils/twill/twill-directive.md`: reorder the procedure so the writer writes the reviewed files and runs the self-check until it passes, then spawns every lens fork in one message, then waits, judges, records and runs the final self-check as today.
- The fork boilerplate: the forks no longer wait on a ready marker; they review the reviewed files as their inherited context holds them, without reading them again, and read only what their context lacks (the existing report line for such reads stays).
- The spawn point per host in `internal/loomengine/twill.go` (Discussion's inline sentence and `planTwillSpawnPoint`): say the procedure runs once the files are written and the self-check passes.
- The ready marker exists only for the forks' wait: drop it, its `twilldirective.Values` field and marker, `DiscussionReadyMarker`/`PlanReadyMarker` and their removal in `internal/loomcli/wiring.go`, unless something else is found to read it.
- The relaunch rules in the directive: a writer that died after writing but before spawning still spawns once; a writer that died after spawning never spawns again, as today.
- Docs: `internal/twilldirective` and `internal/loomengine` `doc.go` where they describe the spawn point or the marker.

## Out of scope

The fork0 arms (a writer fork, fork0 staying alive) stay in `twill-fork-timing` for further benches.
