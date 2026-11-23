# P0 Root: entries under no project

Rendered 2026-11-23 08:15 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Reviews ask the parent instead of settling past held decisions, open BLOCKING findings or an undriven hardener focus**<br>• A held decision finding gates until the parent rules, and a spent budget with BLOCKING findings open escalates to the parent instead of auto-accepting<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`code-bouncer-asks-parent`](design-code-bouncer-asks-parent.md) | darn · solid | Code-Burler-Fix | neve | 14h<br>11h | burler, loom, darn, arbiter, shed |
| T3 | **A beam pull request description is rewritten after each member landing**<br>• After each member landing under a recorded pull request, a describer agent rewrites the beam change description and lym beam describe updates the pull request; no orch is asked<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`beam-pr-describes-itself`](design-beam-pr-describes-itself.md) | darn · light | Darn-Write | neve | 14h<br>13h | dobby, landing |
| T6 | **A note carries depends_on, shown on the board and kept when promoted**<br>• Only a task carries depends_on, so a note that must wait for a running task looks ready to start<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`note-depends-on`](design-note-depends-on.md) | darn · light | Darn-Write | neve | 15m<br>14m | board |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md)<br>[Sketch](decision-later-transition-push-wait.md) | raddle | T5 | 15h<br>13h | batten, fabric |

## Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 14h<br>11h | burler, fabric |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

## Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Drive the four teardown and archive cases the one-repository hardener left undriven**<br>• Killed discard re-run, keep-local tag push, failed step with no kill, adopt --archives over real owner tags | [`harden-fabric-undriven-cases`](design-harden-fabric-undriven-cases.md) | 11h<br>11h | fabric |
| B2 |  | **lym loom goto's stop path may read an unwired reed runner**<br>• Left open at the autonomy-mode landing (30ed1ee58), finding goto-stop-reads-unwired-reed-runner | [`goto-stop-unwired-reed-runner`](design-goto-stop-unwired-reed-runner.md) | 0m<br>0m | loom, reed |
| B3 |  | **ClearReviewSession may keep the PR-Review parent ask after goto**<br>• Left open at the autonomy-mode landing (30ed1ee58), finding clear-review-session-keeps-parent-review-asked | [`clear-review-keeps-parent-ask`](design-clear-review-keeps-parent-ask.md) | 0m<br>0m | landing, loom |
| B4 |  | **Some dobby halt notices lack their autonomy stamp**<br>• Left open at the autonomy-mode landing (30ed1ee58), finding dobby-run-notices-unstamped-autonomy | [`dobby-notices-unstamped-autonomy`](design-dobby-notices-unstamped-autonomy.md) | 0m<br>0m | dobby, orch |
| B5 |  | **Autonomy mode's goto hook and halt-notice stamps have no tests**<br>• Left open at the autonomy-mode landing (30ed1ee58), finding goto-hook-wiring-untested, halt-notice-call-sites-untested | [`autonomy-wiring-untested`](design-autonomy-wiring-untested.md) | 0m<br>0m | loom, orch, testkit |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Config keys nest under the area they govern, named with dots**<br>• loom.yaml spells the area into flat key names (discussion_review_ask_parent, code_max_bounces); nest them as discussion.review_ask_parent | [`config-keys-nested-by-area`](design-config-keys-nested-by-area.md) | 4m<br>4m | config, loom, drift, docs |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>15h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
