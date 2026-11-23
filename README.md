# Board

Rendered 2026-11-23 09:31 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 5d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>1d | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (0)

[`single-repo-hub`](design-single-repo-hub.md) · Age 4d · Owner `neve`

## [P0 Root: entries under no project](project-root.md) (26)

Owner `neve`

### Running (7)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md) | motif | Discussion-Write | neve | 1h<br>23m | config, loom, drift, docs |
| T3 | **A beam pull request description is rewritten after each member landing**<br>• After each member landing under a recorded pull request, a describer agent rewrites the beam change description and lym beam describe updates the pull request; no orch is asked<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`beam-pr-describes-itself`](design-beam-pr-describes-itself.md) | darn · light | Code-Burler-Fix | neve | 15h<br>15h | dobby, landing |
| T7 | **lym loom pr-findings fails: NewParentAlign: Deps.OpenFabric must not be nil**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pr-findings-open-fabric-nil`](design-pr-findings-open-fabric-nil.md) | darn · light | Code-Bouncer | neve | 1h<br>1h | loom, landing |
| T8 | **Run agents ask a question and exit before the reply; drift judge asks in its pane**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`agents-exit-before-reply`](design-agents-exit-before-reply.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | shuttle, creel, drift, stencil |
| T9 | **Plan amendment during Webster loses the record of committed batches**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`amend-plan-loses-webster-batches`](design-amend-plan-loses-webster-batches.md) | darn · light | halted · Code-Bouncer | neve | 1h<br>1h | webster, loom |
| T11 | **tools/godocreflow worsens wrapped-break findings instead of clearing them**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`godocreflow-worsens-findings`](design-godocreflow-worsens-findings.md) | darn · light | Code-Burler-Review | neve | 1h<br>1h | tools, lint |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light | Darn-Write | neve | 12h<br>59m | fabric |

### Ready (7)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Board view lists a done beam-project task under P0**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-done-beam-task-under-p0`](design-board-done-beam-task-under-p0.md) | darn · light |  | 1h<br>1h | board |
| T5 | **RC button shows green while Remote Control is disabled by organization policy**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`rc-button-ignores-org-policy`](design-rc-button-ignores-org-policy.md) | darn · light |  | 42m<br>41m | reed, claude |
| T6 | **Verify-budget notice names no run**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`verify-budget-notice-no-slug`](design-verify-budget-notice-no-slug.md) | darn · light |  | 41m<br>32m | loom |
| T13 | **Decision record written with a CLI error envelope as its Goal**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`record-goal-holds-error`](design-record-goal-holds-error.md) | darn · light |  | 1h<br>1h | loom, darn |
| T14 | **Landing blames the token when the branch-rules read returns a plan-limit 403**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`rules-read-plan-gated-403`](design-rules-read-plan-gated-403.md) | darn · light |  | 1h<br>1h | landing, git |
| T15 | **Find who pushes weft main outside the board push lock**<br>• Board pushes of loomyard-weft refs/heads/main fail with cannot lock ref four times in a row while several runs start<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`weft-main-push-race`](design-weft-main-push-race.md) | darn · docs |  | 1h<br>1h | fabric, board, batten |
| T18 | **Preflight worktree-clean blocks on lym's own _lym/usage session record**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`usage-record-blocks-preflight`](design-usage-record-blocks-preflight.md) | darn · light |  | 14m<br>14m | preflight, fabric |

### Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **lym loom lint-comments --base checks committed HEAD, not the working tree**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`lint-comments-reads-head`](design-lint-comments-reads-head.md) | darn · light | T11 | 1h<br>1h | lint, loom |
| T17 | **Close the four findings autonomy mode landed open**<br>• goto stop reads an unwired reed runner; ClearReviewSession keeps the parent ask; dobby notices lack the autonomy stamp; goto hook and notice stamps untested<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`autonomy-mode-followups`](design-autonomy-mode-followups.md) | darn · light | T7 | 1h<br>59m | loom, landing, dobby, orch, reed, testkit |
| T19 | **Red-first gate holds every run in a hub whose test scoper is not go**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`red-first-non-go-scoper`](design-red-first-non-go-scoper.md) | darn · light | T6 | 14m<br>14m | gate, loom, dotnet |

### Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | `draft` **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | T16<br>T15<br>T8<br>T3<br>T7<br>T9 | 0m<br>0m | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 15h<br>31m | burler, fabric |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

### Bugs (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Narrowed code_auto_accept reaches hubs only as a flag** | [`code-auto-accept-flag-only`](design-code-auto-accept-flag-only.md) | 21m<br>21m | loom, config |
| B2 |  | **Hardener focus-finding wiring is not tested end to end** | [`hardener-focus-wiring-untested`](design-hardener-focus-wiring-untested.md) | 21m<br>21m | loom |
| B3 |  | **Decision accept below the soft-cap window passes an open BLOCKING finding** | [`below-window-accept-blocking`](design-below-window-accept-blocking.md) | 21m<br>21m | loom |
| B4 |  | **Decision-findings detail of a budget escalation has no test** | [`budget-escalation-untested`](design-budget-escalation-untested.md) | 21m<br>21m | loom |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>16h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

### Done (1)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T4 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md)<br>[Sketch](decision-later-transition-push-wait.md) | 16h<br>15h | batten, fabric |
