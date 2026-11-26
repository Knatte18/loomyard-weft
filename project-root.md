# P0 Root: entries under no project

Rendered 2026-11-26 14:14 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Webster run-exit audit flags Master friction note as a parent write, and accept-audit cannot clear it** | [`audit-flags-master-friction`](design-audit-flags-master-friction.md) | darn · light | Code-Burler-Fix | neve | 1h<br>54m | webster |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | Code-Burler-Fix | neve | 3d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Layer A (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Loom status pane not restored after a reboot and a resumed run** | [`resume-restores-status-pane`](design-resume-restores-status-pane.md) | darn · light | T3 | 1h<br>1h | loom, reed, tender |
| T19 | **Drift-Verify reuses a stale loosened ruling after the moved checks land** | [`drift-stale-loosened-ruling`](design-drift-stale-loosened-ruling.md) | darn · light | T3 | 1h<br>1h | drift |
| T20 | **Guard rows: an update mode for the settings golden, and a row that can state an exception** | [`guard-row-tooling`](design-guard-row-tooling.md) | darn · light | T3 | 1h<br>1h | process, claude, gate |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T3<br>T11 | 2h<br>54m | fabric, cleanup, batten |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | T3<br>T4 | 1h<br>1h | gate, testmap |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

## Enhancements (3)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | T3 | 2h<br>2h | burler, stencil |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) |  | 59m<br>59m | slack, creel |
| E3 |  | **Name config keys as <module>.<path> in Go comments** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | T3 | 22m<br>22m | docs |
