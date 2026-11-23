# Board

Rendered 2026-11-23 12:08 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 6d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>1h | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (2)

[`single-repo-hub`](design-single-repo-hub.md) · Age 4d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 18h<br>2h | burler, fabric |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B5 |  | **One-repository fabric cases no hardener round drove** | [`single-repo-undriven-cases`](design-single-repo-undriven-cases.md) | 2h<br>2h | fabric |

## [P0 Root: entries under no project](project-root.md) (18)

Owner `neve`

### Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Intake checks whether an issue is already fixed before it reaches the board** | [`intake-checks-already-fixed`](design-intake-checks-already-fixed.md) | darn · light | Code-Burler-Fix | neve | 2h<br>1h | board |
| T3 | **Start prompt held when a long strand name fills the input box top border** | [`prompt-not-typed-after-skills`](design-prompt-not-typed-after-skills.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | shuttle |
| T5 | **RC button shows green while Remote Control is disabled by organization policy** | [`rc-button-ignores-org-policy`](design-rc-button-ignores-org-policy.md) | darn · light | Code-Burler-Review | neve | 3h<br>3h | reed, claude |
| T6 | **Verify-budget notice names no run** | [`verify-budget-notice-no-slug`](design-verify-budget-notice-no-slug.md) | darn · light | Code-Burler-Review | neve | 3h<br>3h | loom |
| T8 | **Run agents ask a question and exit before the reply; drift judge asks in its pane**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`agents-exit-before-reply`](design-agents-exit-before-reply.md) | darn · light | Describe | neve | 3h<br>3h | shuttle, creel, drift, stencil |
| T16 | **Fix the five fabric faults the one-repository hardener found but could not raise**<br>• Killed clone has no working way forward; merge skips the layout read; kept local tags pushed silently; legacy plain tags unnamed; discard re-run names no command<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | darn · light | Darn-Write | neve | 15h<br>3h | fabric |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Red-first gate holds every run in a hub whose test scoper is not go** | [`red-first-non-go-scoper`](design-red-first-non-go-scoper.md) | darn · light | T6 | 2h<br>2h | gate, loom, dotnet |

### Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | `draft` **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | T16<br>T8 | 2h<br>2h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md)<br>[Sketch](decision-config-keys-nested-by-area.md) | 3h<br>1h | config, loom, drift, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

### Bugs (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Narrowed code_auto_accept reaches hubs only as a flag** | [`code-auto-accept-flag-only`](design-code-auto-accept-flag-only.md) | 2h<br>2h | loom, config |
| B2 |  | **Hardener focus-finding wiring is not tested end to end** | [`hardener-focus-wiring-untested`](design-hardener-focus-wiring-untested.md) | 2h<br>2h | loom |
| B3 |  | **Decision accept below the soft-cap window passes an open BLOCKING finding** | [`below-window-accept-blocking`](design-below-window-accept-blocking.md) | 2h<br>2h | loom |
| B4 |  | **Decision-findings detail of a budget escalation has no test** | [`budget-escalation-untested`](design-budget-escalation-untested.md) | 2h<br>2h | loom |
| B6 |  | **Runs stop and escalate on a transient GitHub fetch failure instead of retrying** | [`transient-fetch-no-retry`](design-transient-fetch-no-retry.md) | 2h<br>2h | git, batten, landing |
| B7 |  | **Every darn escalates to the parent when round 1 fixed a BLOCKING finding** | [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md) | 1h<br>1h | loom |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>18h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
