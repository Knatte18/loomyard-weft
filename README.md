# Board

Rendered 2026-11-26 11:38 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `off` · PR setting `off`.
On the mode line of a beam or the root, a **bold** value is that project's own setting,
and an _italic_ value is inherited from the place in brackets.
A setting that the root and every beam read by this render take from the hub-wide default is stated here alone.
A beam the render could not read, such as one whose worktree is on another machine, has no mode line,
and its settings may differ.
Beam PR links the pull request the beam's landing record names; it may already be closed or merged on GitHub.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (3)

`intake-wave-1` · Age 3h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Webster: plan fixes after begun batches, and recovery that ends asking** | [`webster-plan-recovery`](design-webster-plan-recovery.md) | darn · light | Code-Burler-Review | neve | 4h<br>30m | webster, quarry |

### Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | T3 | 2d<br>2h | landing, loom, config |
| T4 | **Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers** | [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md) | darn · light | T3 | 4h<br>3h | gate, dotnet, darn, burler |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 9d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>3h | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (3)

`intake-wave-2` · Age 3h · Owner `neve`

### Draft (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | T3<br>T1 | 4h<br>3h | landing, stencil, burler, shuttle |
| T10 | `draft` **Machine stops, dead judges and idle seats are not producer failures** | [`infra-stops-not-failures`](design-infra-stops-not-failures.md) | darn · solid | T3<br>T4 | 4h<br>3h | shuttle, claude, tender, gate, burler, shed, reed |
| T11 | `draft` **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | darn · solid | T3<br>T1<br>T9<br>T10 | 4h<br>1h | fabric, shed, landing, batten, dobby |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (0)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (2)

`intake-wave-3` · Age 3h · Owner `neve`

### Draft (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | `draft` **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T3<br>T9<br>T11 | 4h<br>3h | batten, landing, orch, stencil, shed, loom, pawl |
| T13 | `draft` **Close a task's scope at Code-Review, and catch earlier what a PR review found** | [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md) | darn · solid | T3<br>T9<br>T10 | 4h<br>3h | loom, burler |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (3)

`intake-motifs` · Age 3h · Owner `neve`

### Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records`](design-keep-landed-records.md)<br>[Sketch](decision-keep-landed-records.md) | raddle | halted · Worktree-Create | neve | 4h<br>0m | landing, fabric, report |
| T6 | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md)<br>[Sketch](decision-sandbox-closed-area-build.md) | raddle | Raddle-Write | neve | 3w<br>0m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

### Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Dobby run watch and gate-hold notice: mail the parent on every stop and on a slot held too long** | [`dobby-run-watch-build`](design-dobby-run-watch-build.md)<br>[Sketch](decision-dobby-run-watch-build.md) | tapestry · solid | T3<br>T4 | 16m<br>16m | dobby, gate, batten, orch, pattern |

## [P0 Root: entries under no project](project-root.md) (5)

Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | Webster | neve | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

### Bugs (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | T3<br>T11 | 13m<br>13m | fabric, cleanup, batten |

### Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | T3 | 13m<br>13m | burler, stencil |
