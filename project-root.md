# P0 Root: entries under no project

Rendered 2026-11-23 13:47 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Runs stop and escalate on a transient GitHub fetch failure instead of retrying** | [`transient-fetch-no-retry`](design-transient-fetch-no-retry.md) | darn · light | landing · Finalize | neve | 3h<br>1h | git, batten, landing |
| T5 | **Bouncer escalation fixes: fixed BLOCKING findings, soft-cap accept, auto-accept flag, untested wiring** | [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md) | darn · light | Darn-Write | neve | 4h<br>8m | loom, config |
| T6 | **Parent-review asks look up the orch pane in a stale tmux window, so an approved PR review cannot end** | [`parent-review-stale-window`](design-parent-review-stale-window.md) | darn · light | Darn-Write | neve | 1h<br>1h | landing, shuttle |
| T7 | **Webster stops after an amended plan send the parent no notice, and webster reset leaves the run tied to the dropped code tip** | [`webster-amend-stop-no-notice`](design-webster-amend-stop-no-notice.md) | darn · light | Code-Burler-Review | neve | 1h<br>1h | webster, loom |
| T9 | **Red-first gate must support C# repositories** | [`red-first-csharp`](design-red-first-csharp.md) | darn · solid | Darn-Write | neve | 17m<br>8m | gate, loom, dotnet |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | starting · Worktree-Create | neve | 4h<br>4h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **PR-Review row does not notice its session ended** | [`pr-review-dead-session-unseen`](design-pr-review-dead-session-unseen.md) | darn · light | T6 | 17m<br>16m | landing, shuttle |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T1<br>T6<br>T7<br>T10<br>T4<br>T5<br>T9<br>T8 | 21m<br>8m | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>20h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (2)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T2 | **PR-Review re-ask is never delivered: pr-reviewer strand may not sign as the program LOOM** | [`pr-review-reask-not-delivered`](design-pr-review-reask-not-delivered.md) | 1h<br>1h | creel, loom |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | 17h<br>5h | fabric |
