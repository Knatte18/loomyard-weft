# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-27 19:00 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 3h · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Build lym for a gate call only when a targeted package test uses it**<br>• prebuildLym builds lym on every tier1 call, 2.4 to 2.8 s, whether or not a test uses the binary | [`gate-fixed-cost-cuts`](design-gate-fixed-cost-cuts.md) | darn · light | Code-Burler-Fix | neve | 41m<br>36m | gate |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Split the long battencli integration scenario so its parts run in parallel** | [`battencli-split-long-tests`](design-battencli-split-long-tests.md) | darn · light | T11 | 1h<br>1h | batten, testkit |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E6 |  | **TestGateRedFirst_Dotnet sets the floor of the gatecli tier1 time at 31 to 47 s**<br>• After the scenario split, one .NET red-first test holds gatecli tier1 near its old time | [`gatecli-dotnet-redfirst-time`](design-gatecli-dotnet-redfirst-time.md) | 17m<br>17m | gate |
