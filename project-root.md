# P0 Root: entries under no project

Rendered 2026-11-23 14:20 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Bouncer escalation fixes: fixed BLOCKING findings, soft-cap accept, auto-accept flag, untested wiring** | [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md) | darn · light | Code-Burler-Fix | neve | 5h<br>41m | loom, config |
| T6 | **Parent-review asks look up the orch pane in a stale tmux window, so an approved PR review cannot end** | [`parent-review-stale-window`](design-parent-review-stale-window.md) | darn · light | Drift-Verify | neve | 1h<br>1h | landing, shuttle |
| T7 | **Webster stops after an amended plan send the parent no notice, and webster reset leaves the run tied to the dropped code tip** | [`webster-amend-stop-no-notice`](design-webster-amend-stop-no-notice.md) | darn · light | Code-Bouncer | neve | 1h<br>1h | webster, loom |
| T9 | **Red-first gate must support C# repositories** | [`red-first-csharp`](design-red-first-csharp.md) | darn · solid | Darn-Write | neve | 49m<br>41m | gate, loom, dotnet |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | Plan-Burler-Review | neve | 4h<br>4h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **PR-Review row does not notice its session ended** | [`pr-review-dead-session-unseen`](design-pr-review-dead-session-unseen.md) | darn · light | T6 | 50m<br>49m | landing, shuttle |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T6<br>T7<br>T10<br>T4<br>T5<br>T9 | 54m<br>40m | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | 22m<br>22m |  |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>21h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
