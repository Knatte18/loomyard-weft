## Goal

```markdown
## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` found that every tier1 `lym gate test` call pays about 12 s before any test runs, more than the test time in most packages.
It did not time the steps one by one. Time them, so the next cut targets the step that costs.

## Work

- Time each fixed step of a tier1 gate call separately: `go build ./...`, `go vet` of the packages, the build of the `lym` binary, the test binary's link, and anything else the gate runs before the tests.
- Take three cases: one small package alone, ten packages in one call, and the whole tier1 set in one call as a task gate runs it.
- Run each case three times, and record the 1-minute load average and the gate slots held at each start.
- Try two cheap variants once each on the ten-package case: linking without DWARF (`-ldflags=-w`), and skipping `go build ./...` where the tree was already built.
- Read the gate code for each step and say which could run once per tree, or be dropped.

## Output

A report under `docs/research/speed/` with the timings, the variants' effect and one recommended change per step.
Make no code change.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Measurement tiers and gate calls

Decision: The parent added temporary manual tiers tier4 (go build ./... alone) and tier5 (go vet {targets} alone) at the hub default layer, and removes them when the writer reports done. The skip-build variant is derived as tier1 minus tier4 on the same case. The writer may run lym gate test --wait --no-carry with --tier 1, 4 or 5 in the foreground, one call at a time: three runs each for the one-package and ten-package cases, two runs for the whole tier1 set. Link time comes from -debug-actiongraph and the DWARF variant from -ldflags=-w, both passed after --; the lym binary build and other steps come from the gate trace logs.

Rationale: The entry asks for each fixed step timed separately; the writer may not edit hub config or run raw module-wide go commands, and the whole-set calls hold a gate slot for about ten minutes each while other runs need slots.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
