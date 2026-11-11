# Decide how the coverage maps are kept fresh, from the upkeep research

`testmap-upkeep-design` · Task · sketch, testkit, gate, dobby · done

Decision record: [Sketch](decision-testmap-upkeep-design.md)

## Why

The research task `testmap-upkeep` landed `docs/research/speed/testmap-upkeep-2026-11-11/report.md`, which decides nothing.
`testmap-cross-package`, `coverage-test-select` and `gate-scope-coverage` were filed on the premise that this design exists, and `testmap-cross-package` stopped before any code for that reason.

Built since (board tidy, 2026-11-11): `testmap-cross-package` has landed (`99bec44e`).
Its maps record a linked package's coverage by function (`Funcs`, `Linked`, `OtherFiles`, `MapTest.Spawn`, and a `Format` marker with an environment stamp), which agrees with this record's D3.
It left staleness as it was: a change to a linked package's file alone leaves a map fresh, until the build of D1 lands.

## This task

Run Discussion alone over the report and store the decision record on this entry.
Decide:

- whether a change in a linked-only package leaves a map fresh, which the cross-package criterion now assumes;
- whether a failed build keeps the old map or writes a failed-build map;
- whether testmap skips spawn-marked tests;
- whether another package's coverage is recorded by function or by block;
- what the decision changes in `testmap-cross-package`, `coverage-test-select`, `gate-scope-coverage`, `testmap-backlog-drain` and `redundancy-sweep-existing`.

The three build tasks stay held until this record is approved.
