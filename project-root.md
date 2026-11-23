# P0 Root: entries under no project

Rendered 2026-11-23 08:50 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (8)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Reviews ask the parent instead of settling past held decisions, open BLOCKING findings or an undriven hardener focus**<br>• A held decision finding gates until the parent rules, and a spent budget with BLOCKING findings open escalates to the parent instead of auto-accepting<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`code-bouncer-asks-parent`](design-code-bouncer-asks-parent.md) | darn · solid | Code-Bouncer | neve | 14h<br>11h | burler, loom, darn, arbiter, shed |
| T3 | **A beam pull request description is rewritten after each member landing**<br>• After each member landing under a recorded pull request, a describer agent rewrites the beam change description and lym beam describe updates the pull request; no orch is asked<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`beam-pr-describes-itself`](design-beam-pr-describes-itself.md) | darn · light | Darn-Write | neve | 14h<br>14h | dobby, landing |
| T6 | **A note carries depends_on, shown on the board and kept when promoted**<br>• Only a task carries depends_on, so a note that must wait for a running task looks ready to start<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`note-depends-on`](design-note-depends-on.md) | darn · light | Describe | neve | 50m<br>50m | board |
| T7 | **lym loom pr-findings fails: NewParentAlign: Deps.OpenFabric must not be nil**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pr-findings-open-fabric-nil`](design-pr-findings-open-fabric-nil.md) | darn · light | Code-Burler-Review | neve | 31m<br>30m | loom, landing |
| T8 | **Run agents ask a question and exit before the reply; drift judge asks in its pane**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`agents-exit-before-reply`](design-agents-exit-before-reply.md) | darn · light | Darn-Write | neve | 31m<br>30m | shuttle, creel, drift, stencil |
| T9 | **Plan amendment during Webster loses the record of committed batches**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`amend-plan-loses-webster-batches`](design-amend-plan-loses-webster-batches.md) | darn · light | Code-Burler-Review | neve | 31m<br>30m | webster, loom |
| T10 | **Verify gate requires a fail-on-base test for a tests-only task**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`red-first-tests-only-task`](design-red-first-tests-only-task.md) | darn · light | Code-Burler-Review | neve | 31m<br>30m | gate, darn |
| T11 | **tools/godocreflow worsens wrapped-break findings instead of clearing them**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`godocreflow-worsens-findings`](design-godocreflow-worsens-findings.md) | darn · light | Darn-Write | neve | 31m<br>30m | tools, lint |

## Ready (6)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md)<br>[Sketch](decision-later-transition-push-wait.md) | raddle |  | 15h<br>14h | batten, fabric |
| T5 | **RC button shows green while Remote Control is disabled by organization policy**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`rc-button-ignores-org-policy`](design-rc-button-ignores-org-policy.md) | darn · light |  | 0m<br>0m | reed, claude |
| T13 | **Decision record written with a CLI error envelope as its Goal**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`record-goal-holds-error`](design-record-goal-holds-error.md) | darn · light |  | 31m<br>30m | loom, darn |
| T14 | **Landing blames the token when the branch-rules read returns a plan-limit 403**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`rules-read-plan-gated-403`](design-rules-read-plan-gated-403.md) | darn · light |  | 31m<br>30m | landing, git |
| T15 | **Find who pushes weft main outside the board push lock**<br>• Board pushes of loomyard-weft refs/heads/main fail with cannot lock ref four times in a row while several runs start<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`weft-main-push-race`](design-weft-main-push-race.md) | darn · docs |  | 24m<br>24m | fabric, board, batten |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light |  | 12h<br>18m | fabric |

## Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Board view lists a done beam-project task under P0**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-done-beam-task-under-p0`](design-board-done-beam-task-under-p0.md) | darn · light | T6 | 31m<br>30m | board |
| T12 | **lym loom lint-comments --base checks committed HEAD, not the working tree**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`lint-comments-reads-head`](design-lint-comments-reads-head.md) | darn · light | T11 | 31m<br>30m | lint, loom |
| T17 | **Close the four findings autonomy mode landed open**<br>• goto stop reads an unwired reed runner; ClearReviewSession keeps the parent ask; dobby notices lack the autonomy stamp; goto hook and notice stamps untested<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`autonomy-mode-followups`](design-autonomy-mode-followups.md) | darn · light | T1<br>T7 | 35m<br>18m | loom, landing, dobby, orch, reed, testkit |

## Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 14h<br>12h | burler, fabric |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Verify-budget notice names no run** | [`verify-budget-notice-no-slug`](design-verify-budget-notice-no-slug.md) | 0m<br>0m | loom |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md) | 39m<br>39m | config, loom, drift, docs |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>15h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
