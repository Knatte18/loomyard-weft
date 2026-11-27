# P0 Root: entries under no project

Rendered 2026-11-27 06:24 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Desktop launchers are named "lym: <hub>" instead of "lym: start the hub <hub>"** | [`desktop-launcher-names`](design-desktop-launcher-names.md) | darn · light | Darn-Write | neve | 9h<br>8h | desktop |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | landing · Finalize | neve | 17h<br>17h | gate, testmap |
| T19 | **Card gate runs only the card's listed packages, so a wider intent breaks tests unseen** | [`card-gate-scope-gaps`](design-card-gate-scope-gaps.md) | darn · light | Code-Burler-Fix | neve | 10h<br>10h | webster, gate, loom |
| T20 | **Flaky gate: TestGateTestMap/NamedPackageIsRebuiltWhenFresh reports an unexpected PassedAlone entry under load** | [`gatemap-passedalone-flake`](design-gatemap-passedalone-flake.md) | darn · light | Darn-Write | neve | 10h<br>10h | gate |
| T22 | **batten cancel --remove of a beam leaves the project's beam flag set** | [`beam-cancel-keeps-flag`](design-beam-cancel-keeps-flag.md) | darn · light | Darn-Write | neve | 10h<br>10h | batten |

## Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Board README: drop the legend text at the top, and show autonomy and PR setting on P0 and every beam** | [`readme-header-and-mode-lines`](design-readme-header-and-mode-lines.md) | darn · speed |  | 9h<br>8h | board |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 18h<br>17h | fabric, cleanup, batten |

## Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T8<br>T18<br>T21 | 16h<br>10h | docs |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Evaluate the coverage maps (testmap), every user of them and what each gives, and whether to remove them** | [`testmap-worth-evaluation`](design-testmap-worth-evaluation.md) | 9h<br>9h | testmap, gate |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Whole-repo guards run at landing, and the dead-code check with the board tidy, not in every task gate** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) | 9h<br>9h | gate, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) | 17h<br>17h | slack, creel |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs** | [`kde-window-placement`](design-kde-window-placement.md) | 9h<br>9h | termwindow, desktop |
