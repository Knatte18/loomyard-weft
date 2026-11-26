# P0 Root: entries under no project

Rendered 2026-11-26 12:46 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Slack relay: skip its own bridge posts, and give relay commands a prefix** | [`slack-relay-fixes`](design-slack-relay-fixes.md) | darn · solid | landing · Finalize | neve | 30m<br>23m | slack |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | Webster | neve | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |
| T5 | **A tools/ program that measures process, file and new-binary costs, so machines and XDR exceptions can be compared** | [`machine-cost-probe`](design-machine-cost-probe.md) | darn · light | Code-Burler-Review | neve | 2w<br> | tools, gate |

## Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Loom status pane not restored after a reboot and a resumed run** | [`resume-restores-status-pane`](design-resume-restores-status-pane.md) | darn · light | T3 | 16m<br>15m | loom, reed, tender |
| T19 | **Drift-Verify reuses a stale loosened ruling after the moved checks land** | [`drift-stale-loosened-ruling`](design-drift-stale-loosened-ruling.md) | darn · light | T3 | 15m<br>15m | drift |
| T20 | **Guard rows: an update mode for the settings golden, and a row that can state an exception** | [`guard-row-tooling`](design-guard-row-tooling.md) | darn · light | T3 | 16m<br>8m | process, claude, gate |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | T3<br>T4 | 16m<br>15m | gate, testmap |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

## Bugs (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | T3<br>T11 | 1h<br>1h | fabric, cleanup, batten |
| B2 |  | **A decision record criterion with an unindented continuation line is cut at its first line** | [`record-criterion-line-cut`](design-record-criterion-line-cut.md) |  | 58m<br>58m | parsers, loom |

## Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | T3 | 1h<br>1h | burler, stencil |
