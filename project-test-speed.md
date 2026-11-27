# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-27 18:31 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 3h · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Build lym for a gate call only when a targeted package test uses it**<br>• prebuildLym builds lym on every tier1 call, 2.4 to 2.8 s, whether or not a test uses the binary | [`gate-fixed-cost-cuts`](design-gate-fixed-cost-cuts.md) | darn · light | Darn-Write | neve | 12m<br>7m | gate |
| T4 | **Split the long gatecli tier1 scenario tests** | [`gatecli-split-long-tests`](design-gatecli-split-long-tests.md) | darn · light | landing · Finalize | neve | 33m<br>33m | gate, testkit |
| T5 | **Run webstercli and loomcli tier1 tests in parallel and share read-only fixtures** | [`webster-loom-tests-parallel`](design-webster-loom-tests-parallel.md) | darn · light | Code-Burler-Review | neve | 33m<br>33m | webster, loom, testkit |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Split the long battencli integration scenario so its parts run in parallel** | [`battencli-split-long-tests`](design-battencli-split-long-tests.md) | darn · light | T11 | 33m<br>33m | batten, testkit |
