# P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages

Rendered 2026-11-27 17:58 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-speed`](design-test-speed.md) · Age 2h · Owner `neve`

## Aim (operator, 2026-11-27)

Gates are too slow. Measure, package by package, where tier0 and tier1 time goes, then cut or restructure the packages that cost the most, as the fabric-core project does for fabric.

The measurement task comes first; each cut or restructure is filed from its report.
A survey from 2026-11-16 ran under coverage and overstates every time, so it is not used.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Measure the fixed steps of a gate call one by one** | [`gate-fixed-cost-measure`](design-gate-fixed-cost-measure.md) | darn · docs |  | 0m<br>0m | gate |
