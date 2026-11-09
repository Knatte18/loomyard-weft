# P5 Redundant tests found mechanically: a coverage engine, a map per package, a gate report and its review

Rendered 2026-11-09 13:20 UTC; every age below is as of then.
Under: [Board](README.md).

[`redundancy-check`](design-redundancy-check.md) · Age 18h · Owner `osl-1046`

Groups the entries filed from the Sketch `redundant-test-finder`: testcover-engine, testmap-verb, testmap-dobby-refresh, gate-redundancy-report, review-reads-redundancy.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **lym gate testmap rebuilds stale coverage maps in the prime parent's worktree**<br>• `loom: code_assist_reviewer_model astra[medium]` | [`testmap-verb`](design-testmap-verb.md)<br>[Sketch](decision-testmap-verb.md) | darn · solid | Drift-Baseline | osl-1046 | 18h<br>18h | testkit, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **The prime parent's dobby starts lym gate testmap after every test_map_every landings** | [`testmap-dobby-refresh`](design-testmap-dobby-refresh.md)<br>[Sketch](decision-testmap-dobby-refresh.md) | darn · light | T3 | 18h<br>18h | testkit, gate |
| T10 | **An advisory redundancy report step in lym gate test, judging a diff's new tests against the map**<br>• `loom: code_assist_reviewer_model astra[medium]` | [`gate-redundancy-report`](design-gate-redundancy-report.md)<br>[Sketch](decision-gate-redundancy-report.md) | darn · solid | T3 | 18h<br>18h | testkit, gate |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step** | [`review-reads-redundancy`](design-review-reads-redundancy.md)<br>[Sketch](decision-review-reads-redundancy.md) | darn · light | T10 | 18h<br>18h | testkit, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T1 | **internal/testcover: the coverage runner, content block keys and the map record, with testtiming's package-scoped verdict** | [`testcover-engine`](design-testcover-engine.md)<br>[Sketch](decision-testcover-engine.md) | 18h<br>18h | testkit, gate |
