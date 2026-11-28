# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-28 16:13 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 1d · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Measure the test-speed cuts side by side: the packages they changed timed before and after under the same load** | [`test-speed-effect-measure`](design-test-speed-effect-measure.md) | darn · docs | Darn-Write | neve | 1h<br>1h | gate, testkit |
