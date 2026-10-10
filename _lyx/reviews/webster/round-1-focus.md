---
round: 1
focus:
  - "Probe-before-archive ordering in the MultiLLM adapter (card 9) and `seatengine.Engine.Probe` (card 7): settle whether the adapter probes before `archiveStaleOutputs`, whether `Probe` stops every live advisor when the chair is not live, and whether the engine never archives."
  - "Seat stop and join rules in `seatengine` run (card 6): settle whether the chair's turn end alone decides the step, whether advisors are always stopped on every exit path including errors, and whether the per-run send lock and quiet-hold in shuttle (card 1) are held and released on every path."
  - "Reserved markers declared once (cards 4, 5, 11): settle whether the table collision check, the value map and the recipe entry all read `ReservedMarkers`, with no second list."
  - "Stencil include form (card 2) and seat stencils (card 3): settle whether includes are one level deep, markers inside includes are guaranteed, and the shipped seat blocks are registered and marker-pinned as the plan states."
  - "Handle-shaped seams (cards 4, 8, 9, 10): settle whether `shedadapters.SeatRunner` and `seatengine.Shuttle` match the burler shape the plan names, and whether `Env.Seats` and `Env.Models` are filled in `loomCLI.wire` in the same card."
  - "Per-card mechanical checks: for every card, settle whether each group's type-specific check (rename scan, `assert-no-callers`, etc.) actually ran against that group's own targets."
  - "Test economy: settle whether the new `seatengine`, adapter, recipe and fixture tests duplicate each other's coverage (name the covering test), whether per-helper tests repeat what a public-surface test covers, and whether any added behavior has no test; also check `cmd/lyx/testdata/test-budget.yaml` for packages pushed past their row."
  - "Docs in the same commit (card 14 and `doc.go`): settle whether `seatengine` has its module doc, `docs/overview.md` names `seatengine` and the fifth adapter, and `PATTERN.md` carries any new cross-cutting invariant."
---

Seed focus for round 1, drawn from the plan's Shared Decisions and card index.
No entry restates a `Do not flag` item.
