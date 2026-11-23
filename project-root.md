# P0 Root: entries under no project

Rendered 2026-11-23 19:59 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Reed's RC click reads the most recent client's session, not the clicker's** | [`rc-click-wrong-client-session`](design-rc-click-wrong-client-session.md) | darn · light | Darn-Write | neve | 42m<br>42m | reed |
| T2 | **Study: close the bus and tmux exposures without a kernel newer than 7.0** | [`sandbox-without-new-kernel`](design-sandbox-without-new-kernel.md) | darn · speed | Darn-Write | neve | 1h<br>1h | shuttle, reed, claude |
| T4 | **A darn writer ran script -qfc tmux attach outside any memory cap and froze Neve**<br>• `beam: land_notice on` | [`agent-script-tmux-ate-memory`](design-agent-script-tmux-ate-memory.md) | darn · light | Code-Burler-Fix | neve | 36m<br>32m | process, shuttle |
| T5 | **Every agent pane runs in a memory-capped scope, so a runaway child is killed there** | [`agent-pane-memory-cap`](design-agent-pane-memory-cap.md) | darn · light | Darn-Write | neve | 24m<br>24m | reed, shuttle, process |
| T6 | **A run agent killed by a crash or reboot resumes its own session instead of starting fresh**<br>• `beam: land_notice on` | [`resume-every-agent-after-crash`](design-resume-every-agent-after-crash.md) | darn · light | Drift-Await | neve | 11m<br>1m | shuttle, shed |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | Webster | neve | 10h<br>10h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | T10 | 6h<br>5h |  |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T1<br>T2<br>T4<br>T5<br>T10<br>T8<br>T6 | 6h<br>1m | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 6d<br>6d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **No tmux-tier test sends parent-review ask from a live non-orch pane** | [`pr-ask-live-pane-test`](design-pr-ask-live-pane-test.md) | 1h<br>1h | landing, creel |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>42m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
