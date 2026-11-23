# P0 Root: entries under no project

Rendered 2026-11-23 10:14 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (8)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md) | motif | Discussion-Bouncer | neve | 2h<br>1h | config, loom, drift, docs |
| T2 | **Board view lists a done beam-project task under P0**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-done-beam-task-under-p0`](design-board-done-beam-task-under-p0.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | board |
| T8 | **Run agents ask a question and exit before the reply; drift judge asks in its pane**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`agents-exit-before-reply`](design-agents-exit-before-reply.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | shuttle, creel, drift, stencil |
| T11 | **tools/godocreflow worsens wrapped-break findings instead of clearing them**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`godocreflow-worsens-findings`](design-godocreflow-worsens-findings.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | tools, lint |
| T13 | **Decision record written with a CLI error envelope as its Goal** | [`record-goal-holds-error`](design-record-goal-holds-error.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | loom, darn |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light | Darn-Write | neve | 13h<br>1h | fabric |
| T17 | **Close the four findings autonomy mode landed open**<br>• goto stop reads an unwired reed runner; ClearReviewSession keeps the parent ask; dobby notices lack the autonomy stamp; goto hook and notice stamps untested | [`autonomy-mode-followups`](design-autonomy-mode-followups.md) | darn · light | Code-Burler-Review | neve | 1h<br>1h | loom, landing, dobby, orch, reed, testkit |
| T18 | **Preflight worktree-clean blocks on lym's own _lym/usage session record** | [`usage-record-blocks-preflight`](design-usage-record-blocks-preflight.md) | darn · light | starting · Worktree-Create | neve | 57m<br>9m | preflight, fabric |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **lym loom lint-comments --base checks committed HEAD, not the working tree** | [`lint-comments-reads-head`](design-lint-comments-reads-head.md) | darn · light | T11 | 1h<br>1h | lint, loom |
| T19 | **Red-first gate holds every run in a hub whose test scoper is not go** | [`red-first-non-go-scoper`](design-red-first-non-go-scoper.md) | darn · light | T6 | 57m<br>57m | gate, loom, dotnet |

## Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **RC button shows green while Remote Control is disabled by organization policy** | [`rc-button-ignores-org-policy`](design-rc-button-ignores-org-policy.md) | darn · light |  | 1h<br>1h | reed, claude |
| T6 | `draft` **Verify-budget notice names no run** | [`verify-budget-notice-no-slug`](design-verify-budget-notice-no-slug.md) | darn · light |  | 1h<br>1h | loom |
| T10 | `draft` **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | T16<br>T8 | 42m<br>42m | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |
| T14 | `draft` **Landing blames the token when the branch-rules read returns a plan-limit 403** | [`rules-read-plan-gated-403`](design-rules-read-plan-gated-403.md) | darn · light |  | 1h<br>1h | landing, git |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Narrowed code_auto_accept reaches hubs only as a flag** | [`code-auto-accept-flag-only`](design-code-auto-accept-flag-only.md) | 1h<br>1h | loom, config |
| B2 |  | **Hardener focus-finding wiring is not tested end to end** | [`hardener-focus-wiring-untested`](design-hardener-focus-wiring-untested.md) | 1h<br>1h | loom |
| B3 |  | **Decision accept below the soft-cap window passes an open BLOCKING finding** | [`below-window-accept-blocking`](design-below-window-accept-blocking.md) | 1h<br>1h | loom |
| B4 |  | **Decision-findings detail of a budget escalation has no test** | [`budget-escalation-untested`](design-budget-escalation-untested.md) | 1h<br>1h | loom |
| B6 |  | **Runs stop and escalate on a transient GitHub fetch failure instead of retrying** | [`transient-fetch-no-retry`](design-transient-fetch-no-retry.md) | 9m<br>9m | git, batten, landing |
| B7 |  | **Agent sits idle after its skill-load turn; its prompt is never typed** | [`prompt-not-typed-after-skills`](design-prompt-not-typed-after-skills.md) | 1m<br>1m | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Intake checks whether an issue is already fixed before it reaches the board** | [`intake-checks-already-fixed`](design-intake-checks-already-fixed.md) | 15m<br>15m | board |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>16h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (1)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T15 | **Find who pushes weft main outside the board push lock**<br>• Board pushes of loomyard-weft refs/heads/main fail with cannot lock ref four times in a row while several runs start | [`weft-main-push-race`](design-weft-main-push-race.md) | 1h<br>1h | fabric, board, batten |
