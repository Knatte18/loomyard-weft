---
format: 5
approved: true
language: go
---

# Plan: build the coverage maps through `lym gate test --map`

Building the coverage maps becomes a mode of `lym gate test`, set by `--map` over the tier0 and tier1 tests, and the `lym gate testmap` verb goes with its planning, audit round, build locks, waits, `--max` and run-record ordering.
The map run reuses the measure loop of `internal/gatecli/testmappackage.go`: per package one `set`-mode coverage binary, each test alone in its own process, merged into the package's map by `internal/testcover`.
The cards first reroute the loop under the flag and retire the verb, then delete the testcover code only the planning reached, then bound the run by the tiers' parallelism, memory and slots, then add the clean-tree checks, and last reword `lym gate redundant`'s way forward.
The map format, its location and its consumers stay as they are.

## Card Index

1 — gate-test-map-mode — add --map to lym gate test over the scope's packages and retire the testmap verb with its planning, audit, locks, wait and ordering
2 — testcover-planning-delete — delete the testcover selector, upkeep planning and helpers only the retired planning reached
3 — map-run-tier-bounds — run each tier's tests within its parallel and memory bounds, report tests that passed only alone, hold the tiers' slot ask, build no lym and exit non-zero on a failed measurement
4 — map-run-clean-checks — write a map only when HEAD is still and the package's and linked packages' files are clean, listing the rest as unmeasured
5 — redundant-map-way-forward — name lym gate test --map as the way forward of lym gate redundant's map refusals

## Shared Decisions

### Decision: map-run-entry

- **Decision:** `runTest` hands a `--map` request to `runTestMap` right after `flagShapeRefusal`, and `testMapRequest` carries the call as its `scope`.
  The tests keep driving `runTestMap` with `testMapRequest` seams (the go stand-in, the environment and the memory clamp), and the CLI paths go through `newCommand`.
- **Rationale:** the measure loop keeps its seams, and map mode never reaches the tier plan, green record, lints, reports or fixer.
- **Applies to:** cards 1, 3, 4

### Decision: package-set-from-scope

- **Decision:** the package set reads the scope's own shares (`scope.forTier`) and explicit targets, never `scopeInvocations`, takes every package of the module for a whole scope or a whole tier0 or tier1 share, and keeps the packages inside the module of the lowest of tier0 and tier1 that have a map, or that track a Go file at HEAD and hold a tier0 or tier1 test; an explicit target left out is listed under `left_out` with its reason.
- **Rationale:** `scopeInvocations` runs a tier whole when its command holds no targets, and D1 makes map mode ignore the tiers' commands.
- **Applies to:** card 1
