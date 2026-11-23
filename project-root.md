# P0 Root: entries under no project

Rendered 2026-11-23 13:35 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (4)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Runs stop and escalate on a transient GitHub fetch failure instead of retrying** | [`transient-fetch-no-retry`](design-transient-fetch-no-retry.md) | darn · light | Code-Burler-Fix | neve | 3h<br>55m | git, batten, landing |
| T2 | **PR-Review re-ask is never delivered: pr-reviewer strand may not sign as the program LOOM** | [`pr-review-reask-not-delivered`](design-pr-review-reask-not-delivered.md) | darn · light | landing · Finalize | neve | 1h<br>1h | creel, loom |
| T7 | **Webster stops after an amended plan send the parent no notice, and webster reset leaves the run tied to the dropped code tip** | [`webster-amend-stop-no-notice`](design-webster-amend-stop-no-notice.md) | darn · light | Darn-Write | neve | 48m<br>48m | webster, loom |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light | Drift-Recheck | neve | 16h<br>5h | fabric |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Parent-review asks look up the orch pane in a stale tmux window, so an approved PR review cannot end** | [`parent-review-stale-window`](design-parent-review-stale-window.md) | darn · light | T2 | 48m<br>48m | landing, shuttle |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | T16 | 4h<br>4h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **PR-Review row does not notice its session ended** | [`pr-review-dead-session-unseen`](design-pr-review-dead-session-unseen.md) | darn · light | T6 | 5m<br>4m | landing, shuttle |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T1<br>T2<br>T6<br>T7<br>T10<br>T16<br>T4 | 9m<br>4m | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Narrowed code_auto_accept reaches hubs only as a flag** | [`code-auto-accept-flag-only`](design-code-auto-accept-flag-only.md) | 4h<br>4h | loom, config |
| B2 |  | **Hardener focus-finding wiring is not tested end to end** | [`hardener-focus-wiring-untested`](design-hardener-focus-wiring-untested.md) | 4h<br>4h | loom |
| B3 |  | **Decision accept below the soft-cap window passes an open BLOCKING finding** | [`below-window-accept-blocking`](design-below-window-accept-blocking.md) | 4h<br>4h | loom |
| B4 |  | **Decision-findings detail of a budget escalation has no test** | [`budget-escalation-untested`](design-budget-escalation-untested.md) | 4h<br>4h | loom |
| B7 |  | **Every darn escalates to the parent when round 1 fixed a BLOCKING finding** | [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md) | 3h<br>3h | loom |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Red-first gate must support C# repositories** | [`red-first-csharp`](design-red-first-csharp.md) | 5m<br>5m | gate, loom, dotnet |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>20h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
