# P0 Root: entries under no project

Rendered 2026-11-26 20:18 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | darn · light | landing · Finalize | neve | 8h<br>55m | burler, stencil |
| T5 | **Restructure fabricengine into a git-free core so its tests need few repositories** | [`fabric-git-free-core`](design-fabric-git-free-core.md) | motif | Discussion-Burler-Review | neve | 21m<br>15m | fabric, testkit |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | Darn-Write | neve | 7h<br>7h | gate, testmap |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 8h<br>6h | fabric, cleanup, batten |

## Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T8<br>T2<br>T18<br>T21 | 6h<br>0m | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then** | [`enter-session-test-cleanup-race`](design-enter-session-test-cleanup-race.md) | 48m<br>0m | reed |
| B2 |  | **TestLoomPreBootstrap fixture leaves a lym board process running and fails the batch gate** | [`prebootstrap-board-proc-leak`](design-prebootstrap-board-proc-leak.md) | 0m<br>0m | loom, testkit |
| B3 |  | **Card gate runs only the card's listed packages, so a wider intent breaks tests unseen** | [`card-gate-scope-gaps`](design-card-gate-scope-gaps.md) | 0m<br>0m | webster, gate, loom |
| B4 |  | **Flaky gate: TestGateTestMap/NamedPackageIsRebuiltWhenFresh reports an unexpected PassedAlone entry under load** | [`gatemap-passedalone-flake`](design-gatemap-passedalone-flake.md) | 0m<br>0m | gate |
| B5 |  | **batten cancel --remove of a beam leaves the project's beam flag set** | [`beam-cancel-keeps-flag`](design-beam-cancel-keeps-flag.md) | 0m<br>0m | batten |

## Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **PATTERN: tests never fake another module with made-up data** | [`no-fake-other-modules`](design-no-fake-other-modules.md) | 5m<br>5m | pattern, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) | 7h<br>7h | slack, creel |
