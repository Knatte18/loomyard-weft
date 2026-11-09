# P5 Redundant tests found mechanically: a coverage engine, a map per package, a gate report and its review

Rendered 2026-11-09 18:49 UTC; every age below is as of then.
Under: [Board](README.md).

[`redundancy-check`](design-redundancy-check.md) · Age 1d · Owner `neve`

Groups the entries filed from the Sketch `redundant-test-finder`: testcover-engine, testmap-verb, testmap-dobby-refresh, gate-redundancy-report, review-reads-redundancy.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **The prime parent's dobby starts lym gate testmap after every test_map_every landings** | [`testmap-dobby-refresh`](design-testmap-dobby-refresh.md)<br>[Sketch](decision-testmap-dobby-refresh.md) | darn · light |  | 1d<br>1d | testkit, gate |

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | `draft` **An advisory redundancy report step in lym gate test, judging a diff's new tests against the map** | [`gate-redundancy-report`](design-gate-redundancy-report.md)<br>[Sketch](decision-gate-redundancy-report.md) | darn · solid |  | 1d<br>1d | testkit, gate |
| T11 | `draft` **Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step** | [`review-reads-redundancy`](design-review-reads-redundancy.md)<br>[Sketch](decision-review-reads-redundancy.md) | darn · light | T10 | 1d<br>1d | testkit, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T3 | **lym gate testmap rebuilds stale coverage maps in the prime parent's worktree** | [`testmap-verb`](design-testmap-verb.md)<br>[Sketch](decision-testmap-verb.md) | 1d<br>1d | testkit, gate |
