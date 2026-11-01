# Fixer-report, code review round 1

## Changes

- F1 (MEDIUM, fixed as proposed): the long help of `lym orch rules` in `internal/orchcli/rules.go` and the "Orch rules" section of `contracts/stencils/orch/orch-template-role.md` now say a rules file at `_lym/orch/rules.md`, hub-wide on Bolt or per orch, is no longer read, and that its content carries over by copying it to a draft and running `lym orch rules --from`. No migration code. Commit `orch: fix F1`.
- F2 (LOW, fixed as proposed):
  - `internal/heddlecli/cli.go`: `ChangeStore` delegates to a `heddleCLI.changeStore` method, so a test can inject the save and sync seams.
  - `internal/heddlecli/cli_integration_test.go`: subtest `ChangeStoreReturnsErrorsAndWarning` pins the out-of-hub refusal, a refused change and a sync that did not start (warning returned, entry saved).
  - `internal/orchcli/rules_test.go`: row `CwdOutsideAHub` in the refusals test. Commit `heddlecli: fix F2`.
- F2 follow-up from the self-check (below): `writeOrchRules` takes the store-change function as a parameter (`heddlecli.ChangeStore` in production), and the new `TestWriteOrchRules_ReportsTheStoreOutcome` pins the envelope's `warning`, its absence on a clean save and the store-error refusal. Commit `orchcli: fix F2 self-check`.

Tier 0 of `./internal/orchcli` and `./contracts/stencils`, and tier 1 of `./internal/orchcli` and `./internal/heddlecli`, pass.

## Self-check

- fix-code-correctness, LOW: no test reaches the `warning` copy in `writeOrchRules`; fixed as proposed in the follow-up above, with an injected store-change function.
- fix-consistency: no findings.
- fix-proposal: no findings (it noted `./contracts/stencils` tests were not run; run since, tier 0 passes).
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
