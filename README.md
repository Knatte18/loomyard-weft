# Board

Rendered 2026-11-26 21:17 UTC; every age below is as of then.
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

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (1)

`intake-wave-1` · Age 12h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | Code-Burler-Fix | neve | 3d<br>11h | landing, loom, config |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 9d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>1h | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (3)

`intake-wave-2` · Age 12h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T10 | **Machine stops, dead judges and idle seats are not producer failures** | [`infra-stops-not-failures`](design-infra-stops-not-failures.md) | darn · solid | Code-Burler-Review | neve | 13h<br>12h | shuttle, claude, tender, gate, burler, shed, reed |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | T1 | 13h<br>8h | landing, stencil, burler, shuttle |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | tapestry | T1<br>T9<br>T10<br>T5 | 13h<br>29m | fabric, shed, landing, batten, dobby |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (2)

`intake-wave-3` · Age 12h · Owner `neve`

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Close a task's scope at Code-Review, and catch earlier what a PR review found** | [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md) | darn · solid | T9<br>T10 | 13h<br>12h | loom, burler |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T9<br>T11 | 13h<br>8h | batten, landing, orch, stencil, shed, loom, pawl |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (2)

`intake-motifs` · Age 12h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **Dobby run watch and gate-hold notice: mail the parent on every stop and on a slot held too long** | [`dobby-run-watch-build`](design-dobby-run-watch-build.md)<br>[Sketch](decision-dobby-run-watch-build.md) | tapestry · solid | Webster | neve | 9h<br>9h | dobby, gate, batten, orch, pattern |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T1<br>T9<br>T11<br>T12<br>T13 | 9h<br>9h | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 9h · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 9h<br>9h | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T10<br>T8<br>T11 | 9h<br>9h | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 9h<br>9h | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 9h<br>9h | termwindow, hibernate, batten, ide, deploy, reed |

## [P0 Root: entries under no project](project-root.md) (13)

Owner `neve`

### Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **PATTERN: tests never fake another module with made-up data** | [`no-fake-other-modules`](design-no-fake-other-modules.md) | darn · docs | Darn-Write | neve | 1h<br>25m | pattern, testkit |
| T4 | **TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then** | [`enter-session-test-cleanup-race`](design-enter-session-test-cleanup-race.md) | darn · light | landing · Finalize | neve | 1h<br>58m | reed |
| T5 | **Restructure fabricengine into a git-free core so its tests need few repositories** | [`fabric-git-free-core`](design-fabric-git-free-core.md)<br>[Sketch](decision-fabric-git-free-core.md) | raddle | starting · Worktree-Create | neve | 1h<br>0m | fabric, testkit |
| T18 | **gate reach: functions called directly by tier1 tests are reported as reached by no mapped test** | [`reach-misses-tier1-calls`](design-reach-misses-tier1-calls.md) | darn · light | Code-Burler-Review | neve | 8h<br>8h | gate, testmap |
| T19 | **Card gate runs only the card's listed packages, so a wider intent breaks tests unseen** | [`card-gate-scope-gaps`](design-card-gate-scope-gaps.md) | darn · light | Code-Bouncer | neve | 58m<br>58m | webster, gate, loom |

### Ready (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T20 | **Flaky gate: TestGateTestMap/NamedPackageIsRebuiltWhenFresh reports an unexpected PassedAlone entry under load** | [`gatemap-passedalone-flake`](design-gatemap-passedalone-flake.md) | darn · light |  | 58m<br>58m | gate |
| T22 | **batten cancel --remove of a beam leaves the project's beam flag set** | [`beam-cancel-keeps-flag`](design-beam-cancel-keeps-flag.md) | darn · light |  | 58m<br>58m | batten |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 9h<br>7h | fabric, cleanup, batten |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T8<br>T18<br>T21 | 7h<br>58m | docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Evaluate the coverage maps (testmap), every user of them and what each gives, and whether to remove them** | [`testmap-worth-evaluation`](design-testmap-worth-evaluation.md) | 0m<br>0m | testmap, gate |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Whole-repo guards run at landing, and the dead-code check with the board tidy, not in every task gate** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) | 5m<br>3m | gate, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) | 8h<br>8h | slack, creel |

### Done (1)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T6 | **TestLoomPreBootstrap fixture leaves a lym board process running and fails the batch gate** | [`prebootstrap-board-proc-leak`](design-prebootstrap-board-proc-leak.md) | 58m<br>58m | loom, testkit |
