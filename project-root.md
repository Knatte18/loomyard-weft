# P0 Root: entries under no project

Rendered 2026-11-23 13:03 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (8)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Runs stop and escalate on a transient GitHub fetch failure instead of retrying** | [`transient-fetch-no-retry`](design-transient-fetch-no-retry.md) | darn · light | Code-Burler-Review | neve | 2h<br>23m | git, batten, landing |
| T2 | **PR-Review re-ask is never delivered: pr-reviewer strand may not sign as the program LOOM** | [`pr-review-reask-not-delivered`](design-pr-review-reask-not-delivered.md) | darn · light | Code-Burler-Fix | neve | 43m<br>43m | creel, loom |
| T3 | **PR body's appended regression block uses internal task wording** | [`pr-regression-block-wording`](design-pr-regression-block-wording.md) | darn · light | Code-Bouncer | neve | 17m<br>16m | drift, landing |
| T4 | **Driver on older lym build hard-fails a run when the board's stencil is newer** | [`old-driver-fails-newer-stencil`](design-old-driver-fails-newer-stencil.md) | darn · light | Describe | neve | 43m<br>43m | stencil, loom |
| T5 | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md)<br>[Sketch](decision-config-keys-nested-by-area.md) | raddle | Raddle-Write | neve | 4h<br>9m | config, loom, drift, docs |
| T7 | **Webster stops after an amended plan send the parent no notice, and webster reset leaves the run tied to the dropped code tip** | [`webster-amend-stop-no-notice`](design-webster-amend-stop-no-notice.md) | darn · light | Darn-Write | neve | 16m<br>16m | webster, loom |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light | Code-Burler-Fix | neve | 16h<br>4h | fabric |
| T19 | **Red-first gate holds every run in a hub whose test scoper is not go** | [`red-first-non-go-scoper`](design-red-first-non-go-scoper.md) | darn · light | Code-Burler-Fix | neve | 3h<br>3h | gate, loom, dotnet |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Parent-review asks look up the orch pane in a stale tmux window, so an approved PR review cannot end** | [`parent-review-stale-window`](design-parent-review-stale-window.md) | darn · light | T2 | 16m<br>16m | landing, shuttle |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | T16 | 3h<br>3h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Narrowed code_auto_accept reaches hubs only as a flag** | [`code-auto-accept-flag-only`](design-code-auto-accept-flag-only.md) | 3h<br>3h | loom, config |
| B2 |  | **Hardener focus-finding wiring is not tested end to end** | [`hardener-focus-wiring-untested`](design-hardener-focus-wiring-untested.md) | 3h<br>3h | loom |
| B3 |  | **Decision accept below the soft-cap window passes an open BLOCKING finding** | [`below-window-accept-blocking`](design-below-window-accept-blocking.md) | 3h<br>3h | loom |
| B4 |  | **Decision-findings detail of a budget escalation has no test** | [`budget-escalation-untested`](design-budget-escalation-untested.md) | 3h<br>3h | loom |
| B7 |  | **Every darn escalates to the parent when round 1 fixed a BLOCKING finding** | [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md) | 2h<br>2h | loom |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>19h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
