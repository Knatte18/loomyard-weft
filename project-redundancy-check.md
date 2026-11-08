# P5 Redundant tests found mechanically: a coverage engine, a map per package, a gate report and its review

Rendered 2026-11-08 19:02 UTC; every age below is as of then.
Under: [Board](README.md).

[`redundancy-check`](design-redundancy-check.md) · Age 15m · Owner `neve`

Groups the entries filed from the Sketch `redundant-test-finder`: testcover-engine, testmap-verb, testmap-dobby-refresh, gate-redundancy-report, review-reads-redundancy.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **internal/testcover: the coverage runner, content block keys and the map record, with testtiming's package-scoped verdict** | [`testcover-engine`](design-testcover-engine.md)<br>[Sketch](decision-testcover-engine.md) | darn · solid |  | 15m<br>15m | testkit, gate |
| T3 | `draft` **lym gate testmap rebuilds stale coverage maps in the prime parent's worktree** | [`testmap-verb`](design-testmap-verb.md)<br>[Sketch](decision-testmap-verb.md) | darn · solid | T1 | 15m<br>15m | testkit, gate |
| T9 | `draft` **The prime parent's dobby starts lym gate testmap after every test_map_every landings** | [`testmap-dobby-refresh`](design-testmap-dobby-refresh.md)<br>[Sketch](decision-testmap-dobby-refresh.md) | darn · light | T3 | 15m<br>15m | testkit, gate |
| T10 | `draft` **An advisory redundancy report step in lym gate test, judging a diff's new tests against the map** | [`gate-redundancy-report`](design-gate-redundancy-report.md)<br>[Sketch](decision-gate-redundancy-report.md) | darn · solid | T3 | 15m<br>15m | testkit, gate |
| T11 | `draft` **Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step** | [`review-reads-redundancy`](design-review-reads-redundancy.md)<br>[Sketch](decision-review-reads-redundancy.md) | darn · light | T10 | 15m<br>15m | testkit, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
