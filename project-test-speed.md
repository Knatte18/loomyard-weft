# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-27 18:15 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 2h · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Measure the fixed steps of a gate call one by one** | [`gate-fixed-cost-measure`](design-gate-fixed-cost-measure.md) | darn · docs | Drift-Verify | neve | 17m<br>17m | gate |
| T4 | **Split the long gatecli tier1 scenario tests** | [`gatecli-split-long-tests`](design-gatecli-split-long-tests.md) | darn · light | Darn-Write | neve | 17m<br>17m | gate, testkit |
| T5 | **Run webstercli and loomcli tier1 tests in parallel and share read-only fixtures** | [`webster-loom-tests-parallel`](design-webster-loom-tests-parallel.md) | darn · light | Darn-Write | neve | 17m<br>17m | webster, loom, testkit |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Split the long battencli integration scenario so its parts run in parallel** | [`battencli-split-long-tests`](design-battencli-split-long-tests.md) | darn · light | T11 | 17m<br>17m | batten, testkit |
