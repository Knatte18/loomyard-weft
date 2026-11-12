# Measure what coverage-map upkeep costs, check how its state is shown, and write the operator guide

`testmap-upkeep-cost-and-guide` · Task · research, gate, testkit, dobby · draft

Depends on: [`testmap-maps-in-weft`](design-testmap-maps-in-weft.md)

## Idea (operator, 2026-11-12)

The operator wants good control over how the coverage maps are used and kept.
The upkeep pass runs on its own now (`test_map_upkeep` on, the dobby starts it), so what it costs and what it shows must be readable at any time.
The pilot (`docs/research/speed/coverage-map-pilot-2026-11-12/report.md`) estimated a full build at 45 to 165 minutes; the first real pass took 44 minutes on one slot.

## What it does

1. Cost: from the pass records and the gate's run records, report what the upkeep passes have cost since `36621f0e` landed:
   passes started, wall time and gate-slot time per pass, packages measured, skipped and failed, the time a pass spent yielding to gates, and the audit mismatches.
   Say how long a typical pass after one landing takes, and how long a full rebuild takes now.
2. State: check whether one call shows the current state of the maps: which are at HEAD, which are queued and why, the miss counts by reason, the audit mismatches and the last pass.
   If none does, propose the smallest way to show it (an existing verb with a flag, or a new read-only verb) with its size as a recipe and profile; write none of it.
3. Use: write a short operator guide to `docs/research/speed/` (and link it from the research README):
   how the maps are kept, when a map is rebuilt whole or by test, how to rebuild one package by hand, how to list candidates (`lym gate redundant`), how to switch the upkeep off and on, and how to tell that something is wrong.

## Cap

Reading existing records and reports; no new full build, no repeat runs, no code change.

## Not in scope

Cutting or folding any test, the selector's miss rules, and building the status view.
