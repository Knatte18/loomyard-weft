# Board

Rendered 2026-11-26 12:36 UTC; every age below is as of then.
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

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (2)

`intake-wave-1` · Age 4h · Owner `neve`

### Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | T3 | 2d<br>3h | landing, loom, config |
| T4 | **Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers** | [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md) | darn · light | T3 | 5h<br>4h | gate, dotnet, darn, burler |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 9d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>4h | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (3)

`intake-wave-2` · Age 4h · Owner `neve`

### Draft (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | T3<br>T1 | 5h<br>5m | landing, stencil, burler, shuttle |
| T10 | `draft` **Machine stops, dead judges and idle seats are not producer failures** | [`infra-stops-not-failures`](design-infra-stops-not-failures.md) | darn · solid | T3<br>T4 | 5h<br>4h | shuttle, claude, tender, gate, burler, shed, reed |
| T11 | `draft` **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | darn · solid | T3<br>T1<br>T9<br>T10 | 5h<br>5m | fabric, shed, landing, batten, dobby |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (0)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (2)

`intake-wave-3` · Age 4h · Owner `neve`

### Draft (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | `draft` **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T3<br>T9<br>T11 | 5h<br>5m | batten, landing, orch, stencil, shed, loom, pawl |
| T13 | `draft` **Close a task's scope at Code-Review, and catch earlier what a PR review found** | [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md) | darn · solid | T3<br>T9<br>T10 | 5h<br>4h | loom, burler |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (3)

`intake-motifs` · Age 4h · Owner `neve`

### [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 37m · Owner `neve`

#### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid | T3 | 37m<br>37m | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T3<br>T10<br>T8<br>T11 | 37m<br>32m | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 37m<br>37m | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 37m<br>37m | termwindow, hibernate, batten, ide, deploy, reed |

### Own entries

#### Draft (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Dobby run watch and gate-hold notice: mail the parent on every stop and on a slot held too long** | [`dobby-run-watch-build`](design-dobby-run-watch-build.md)<br>[Sketch](decision-dobby-run-watch-build.md) | tapestry · solid | T3<br>T4 | 1h<br>1h | dobby, gate, batten, orch, pattern |
| T17 | `draft` **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T3<br>T1<br>T9<br>T11<br>T12<br>T13 | 37m<br>32m | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P0 Root: entries under no project](project-root.md) (12)

Owner `neve`

### Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Slack relay: skip its own bridge posts, and give relay commands a prefix** | [`slack-relay-fixes`](design-slack-relay-fixes.md) | darn · solid | Code-Burler-Review | neve | 19m<br>12m | slack |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | Webster | neve | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |
| T5 | **A tools/ program that measures process, file and new-binary costs, so machines and XDR exceptions can be compared** | [`machine-cost-probe`](design-machine-cost-probe.md) | darn · light | Darn-Write | neve | 2w<br> | tools, gate |

### Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Loom status pane not restored after a reboot and a resumed run** | [`resume-restores-status-pane`](design-resume-restores-status-pane.md) | darn · light | T3 | 5m<br>4m | loom, reed, tender |
| T19 | **Drift-Verify reuses a stale loosened ruling after the moved checks land** | [`drift-stale-loosened-ruling`](design-drift-stale-loosened-ruling.md) | darn · light | T3 | 5m<br>4m | drift |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | T3<br>T4 | 5m<br>4m | gate, testmap |

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

### Bugs (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | T3<br>T11 | 1h<br>1h | fabric, cleanup, batten |
| B2 |  | **A decision record criterion with an unindented continuation line is cut at its first line** | [`record-criterion-line-cut`](design-record-criterion-line-cut.md) |  | 47m<br>47m | parsers, loom |

### Enhancements (3)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Hardener rounds raise what they drive and converge on one rule** | [`hardener-recipe-changes`](design-hardener-recipe-changes.md) | T3 | 1h<br>1h | burler, stencil |
| E2 |  | **Claude settings golden has no update flag** | [`settings-golden-update-flag`](design-settings-golden-update-flag.md) |  | 5m<br>5m | claude, gate |
| E3 |  | **shellguard: a row cannot state an exception, so negation is hand-built in regex** | [`shellguard-row-exception`](design-shellguard-row-exception.md) |  | 5m<br>5m | process |
