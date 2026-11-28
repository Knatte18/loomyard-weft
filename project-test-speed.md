# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-28 12:34 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 21h · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Done (1)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T20 | **TestGateRedFirst_Dotnet sets the floor of the gatecli tier1 time at 31 to 47 s**<br>• After the scenario split, one .NET red-first test holds gatecli tier1 near its old time | [`gatecli-dotnet-redfirst-time`](design-gatecli-dotnet-redfirst-time.md) | 17h<br>3h | gate |
