# Board

Rendered 2026-11-26 08:53 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (6)

`intake-wave-1` · Age 34m · Owner `neve`

### Draft (6)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | T3 | 2d<br>32m | landing, loom, config |
| T2 | `draft` **Webster: plan fixes after begun batches, and recovery that ends asking** | [`webster-plan-recovery`](design-webster-plan-recovery.md) | darn · light | T3 | 1h<br>33m | webster, quarry |
| T4 | `draft` **Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers** | [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md) | darn · light | T3 | 1h<br>33m | gate, dotnet, darn, burler |
| T5 | `draft` **Reed reads each client's own session, and shellguard refuses the session bus** | [`reed-client-and-bus-guard`](design-reed-client-and-bus-guard.md) | darn · light | T3 | 2d<br>33m | reed, process |
| T6 | `draft` **creel post refusal after a persisted cd does not say why** | [`creel-refusal-after-cd`](design-creel-refusal-after-cd.md) | darn · light | T3 | 1h<br> | creel, cwd |
| T7 | `draft` **Iterative agents should commit often** | [`writers-commit-each-step`](design-writers-commit-each-step.md) | darn · light | T3 | 1h<br> | stencil, darn |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 8d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>51m | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (3)

`intake-wave-2` · Age 34m · Owner `neve`

### Draft (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | T3<br>T1 | 1h<br>33m | landing, stencil, burler, shuttle |
| T10 | `draft` **Machine stops, dead judges and idle seats are not producer failures** | [`infra-stops-not-failures`](design-infra-stops-not-failures.md) | darn · solid | T3<br>T4<br>T5 | 1h<br>33m | shuttle, claude, tender, gate, burler, shed, reed |
| T11 | `draft` **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | darn · solid | T3<br>T1<br>T9<br>T10 | 1h<br>0m | fabric, shed, landing, batten, dobby |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 3d<br>2d | burler, fabric |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (2)

`intake-wave-3` · Age 34m · Owner `neve`

### Draft (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | `draft` **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T3<br>T9<br>T11<br>T7 | 1h<br>33m | batten, landing, orch, stencil, shed, loom, pawl |
| T13 | `draft` **Close a task's scope at Code-Review, and catch earlier what a PR review found** | [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md) | darn · solid | T3<br>T9<br>T10 | 1h<br>33m | loom, burler |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (3)

`intake-motifs` · Age 34m · Owner `neve`

### Draft (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T14 | `draft` **Built-in run watch: mail the parent on every non-running state and on a gate slot held too long** | [`dobby-run-watch`](design-dobby-run-watch.md) | motif | T3 | 1h<br>32m | dobby, gate |
| T15 | `draft` **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | motif | T3 | 3w<br>32m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| T16 | `draft` **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records`](design-keep-landed-records.md) | motif | T3 | 1h<br>32m | landing, fabric, report |

## [P0 Root: entries under no project](project-root.md) (4)

Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | Drift-Recheck | neve | 2d<br>2d |  |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T8 | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |
