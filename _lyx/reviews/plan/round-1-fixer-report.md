# Fixer report: plan, round 1

All four findings were validated against the code and the decision record, and all four are fixed.
`lyx loom validate-plan` passes over the edited plan.

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  The package doc sits above `package main` in `tools/tokencount/main.go`, outside the `run` function at `main.go:169`, so no target on card 2 held it.
  Card 2's `**Edit:**` now lists `tools/tokencount/main.go` in place of `tools/tokencount#run`.
  No other member on card 2 resolves into `main.go`: `CountRun`, `RunTally` and `Report.WriteMarkdown` are in `count.go`, so `redundant-file-target` does not fire.
- **F2 (LOW), fixed with the review's first option.**
  The premise holds: `seedAndCommitBootstrap` reads and writes only `shedrun.SelfRunID`'s seed (`internal/loomcli/sharedbootstrap.go`, the `ReadSeed`/`WriteSeed` pair in step 1b), so a parent typed into another run-id's seed is never checked.
  Card 4 now renders the driverless route only for `run` and `step` addressing `self`.
  Every other run-id, and `status`, `pause` and `goto`, keep the `lyx loom start` remedy.
  The reachability sentence, the `seed missing` spec-row split and `TestResolveRunID`'s rows (a new non-`self` `step` row asserting no `lyx shed seed`) follow.
  This drops the card's unlicensed non-`self` rendering instead of documenting it, since D3 and the acceptance criteria name only the `self` form.
- **F3 (LOW), fixed with the review's first option.**
  Card 5 lists `internal/loomcli/smoke_bootstrapwiring_test.go` under `**Edit:**`.
  `TestSmokeStepBootstrapWiring` gains a `wrong-parent seed is cleared and re-seeded` subtest on its own `newWiredPairFixture` with `registerBootstrapTeardown`.
  The subtest follows the way forward (`rm` the seed, `lyx shed seed self … parent=<recorded parent>`) and asserts the next `lyx loom step` succeeds and dispatches Preflight.
  The integration subtest in `TestLoomFailedReedUpRefuses` now asserts only the refusal's text and kind.
  The follow-through moved to the tmux tier and is not duplicated in both.
  No new top-level test, so `internal/loomcli`'s budget row is unaffected by card 5.
- **F4 (NIT), fixed as proposed.**
  Card 3's `docs/overview.md` moves into a `**Prosa:**` group, per the plan spec's Card granularity section, which names `docs/overview.md` as a `Prosa` target.
  The card keeps its `Edit` group and its `ImpactSummary`.

## Disputed

None.

## Deferred

None.
