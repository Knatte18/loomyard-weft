## Goal

```markdown
## Why

The test-speed project's cuts have landed (parallel scenario rows in gatecli, battencli, webstercli and loomcli, the prebuilt lym built only when a test may use it, parallel .NET red-first rows, PR-Review tests over real reed).
The operator asked whether they had an effect (2026-11-28).
The machine will not be quiet for a long time, so the before and after sides run interleaved, under the same load.

## Runs

Exactly these, and no more:

- Sides: A is the tree of `ea00087ec` (before the cuts), B is the tree of `main` at the run's start.
- Packages: `internal/gatecli`, `internal/battencli`, `internal/webstercli`, `internal/loomcli`, `cmd/lym`.
- For each package, tier0 and tier1 through `lym gate test --tier`, in the order A, B, A, B: two repeats per side, side by side in time.
- Side A runs in one extra worktree pair: ask the orch, which creates it from this task's worktree; commit A's tree on top of its branch there.
  Do not push it; tell the orch when it can be removed.
- Record the load average at each call's start.

## Report

`docs/research/speed/test-speed-effect-2026-11-28/report.md`, beside the earlier speed reports:
wall time per package, tier and side, each repeat and the mean, the load at each call, and B against A as minutes and as a share.
Name where a cut shows and where it does not, and how the load limits what the numbers say.
It compares with `ea00087ec`'s package report only to name the packages; the earlier numbers ran under other load.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-28): One repeat per side: 20 gate calls

Decision: Each package runs tier0 and tier1 once per side, in the order A then B: 20 calls, not 40. The spread across the five packages stands in for repeats.

Rationale: Operator, 2026-11-28: 40 calls hold a gate slot for hours; two repeats are not needed.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
