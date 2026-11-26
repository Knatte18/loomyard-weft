# P0 Root: entries under no project

Rendered 2026-11-26 21:11 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then** | [`enter-session-test-cleanup-race`](design-enter-session-test-cleanup-race.md) | darn · light | landing · Finalize | neve | 1h<br>53m | reed |
| T5 | **Restructure fabricengine into a git-free core so its tests need few repositories** | [`fabric-git-free-core`](design-fabric-git-free-core.md) | motif | Discussion-Bouncer | neve | 1h<br>1h | fabric, testkit |
| T6 | **TestLoomPreBootstrap fixture leaves a lym board process running and fails the batch gate** | [`prebootstrap-board-proc-leak`](design-prebootstrap-board-proc-leak.md) | darn · light | landing · Finalize | neve | 53m<br>53m | loom, testkit |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | Darn-Write | neve | 8h<br>8h | gate, testmap |
| T19 | **Card gate runs only the card's listed packages, so a wider intent breaks tests unseen** | [`card-gate-scope-gaps`](design-card-gate-scope-gaps.md) | darn · light | Darn-Write | neve | 53m<br>52m | webster, gate, loom |

## Ready (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **PATTERN: tests never fake another module with made-up data** | [`no-fake-other-modules`](design-no-fake-other-modules.md) | darn · docs |  | 59m<br>20m | pattern, testkit |
| T20 | **Flaky gate: TestGateTestMap/NamedPackageIsRebuiltWhenFresh reports an unexpected PassedAlone entry under load** | [`gatemap-passedalone-flake`](design-gatemap-passedalone-flake.md) | darn · light |  | 53m<br>52m | gate |
| T22 | **batten cancel --remove of a beam leaves the project's beam flag set** | [`beam-cancel-keeps-flag`](design-beam-cancel-keeps-flag.md) | darn · light |  | 53m<br>52m | batten |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 9h<br>7h | fabric, cleanup, batten |

## Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T8<br>T18<br>T21 | 7h<br>52m | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) | 7h<br>7h | slack, creel |
