# P0 Root: entries under no project

Rendered 2026-11-26 08:20 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | Drift-Verify | neve | 2d<br>2d |  |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T8 | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T14 | `draft` **Built-in run watch: mail the parent on every non-running state and on a gate slot held too long** | [`dobby-run-watch`](design-dobby-run-watch.md) | darn | T3 | 46m<br>0m | dobby, gate |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records-in-weft`](design-keep-landed-records-in-weft.md) | 46m<br>46m | landing, fabric, report |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>10m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
