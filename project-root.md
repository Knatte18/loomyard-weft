# P0 Root: entries under no project

Rendered 2026-11-26 20:16 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | darn · light | landing · Finalize | neve | 8h<br>52m | burler, stencil |
| T5 | **Restructure fabricengine into a git-free core so its tests need few repositories** | [`fabric-git-free-core`](design-fabric-git-free-core.md) | motif | Discussion-Bouncer | neve | 19m<br>13m | fabric, testkit |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | Darn-Write | neve | 7h<br>7h | gate, testmap |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 8h<br>6h | fabric, cleanup, batten |

## Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T8<br>T2<br>T18<br>T21 | 6h<br>58m | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then** | [`enter-session-test-cleanup-race`](design-enter-session-test-cleanup-race.md) | 46m<br>46m | reed |

## Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **PATTERN: tests never fake another module with made-up data** | [`no-fake-other-modules`](design-no-fake-other-modules.md) | 3m<br>3m | pattern, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) | 7h<br>7h | slack, creel |
