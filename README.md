# Board

Rendered 2026-11-27 07:51 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `off` · PR setting `off`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (0)

`intake-wave-1` · Age 23h · Owner `neve`

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 9d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 6w<br>19m | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (2)

`intake-wave-2` · Age 23h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | Darn-Write | neve | 1d<br>19h | landing, stencil, burler, shuttle |

### Layer G (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | tapestry | T9<br>T28 | 1d<br>10h | fabric, shed, landing, batten, dobby |

## [P4 Fabric split into a git-free core, a layout package and a thin git engine](project-fabric-core.md) (7)

[`fabric-core`](design-fabric-core.md) · Age 10h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Fabric core foundation: layout and core packages, snapshot, action list, executor loop and the layout retarget** | [`fabric-core-foundation`](design-fabric-core-foundation.md)<br>[Sketch](decision-fabric-core-foundation.md) | tapestry · solid | Plan-Burler-Fix | neve | 10h<br>10h | fabric, testkit, pattern, cwd, loom, batten, landing, gate, orch, board, webster, drift, report |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T23 | **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | T6 | 10h<br>10h | fabric, testkit, shed, board, pattern |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T24 | **Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core** | [`fabric-core-teardown`](design-fabric-core-teardown.md)<br>[Sketch](decision-fabric-core-teardown.md) | tapestry · solid | T23 | 10h<br>10h | fabric, testkit, shed, landing |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T25 | **Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core** | [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)<br>[Sketch](decision-fabric-core-merge-landing.md) | tapestry · solid | T24 | 10h<br>10h | fabric, testkit, shed, batten, landing |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T26 | **Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric** | [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)<br>[Sketch](decision-fabric-core-pull-replay.md) | tapestry · solid | T25 | 10h<br>10h | fabric, testkit, shed, stencil, board, heddle |

### Layer E (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T27 | **Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut** | [`fabric-core-add-clone`](design-fabric-core-add-clone.md)<br>[Sketch](decision-fabric-core-add-clone.md) | tapestry · solid | T26 | 10h<br>10h | fabric, testkit, ide, batten, heddle, reconcile, shed |

### Layer F (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T28 | **Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure** | [`fabric-core-records-guards`](design-fabric-core-records-guards.md)<br>[Sketch](decision-fabric-core-records-guards.md) | tapestry · solid | T27 | 10h<br>10h | fabric, testkit, pattern, docs |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (2)

`intake-wave-3` · Age 23h · Owner `neve`

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Close a task's scope at Code-Review, and catch earlier what a PR review found** | [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md) | darn · solid | T9 | 1d<br>23h | loom, burler |

### Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T9<br>T11 | 1d<br>19h | batten, landing, orch, stencil, shed, loom, pawl |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (1)

`intake-motifs` · Age 23h · Owner `neve`

### Layer I (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T9<br>T11<br>T12<br>T13 | 19h<br>19h | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 19h · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 19h<br>19h | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T11 | 19h<br>19h | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 19h<br>19h | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 19h<br>19h | termwindow, hibernate, batten, ide, deploy, reed |

## [P0 Root: entries under no project](project-root.md) (17)

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

### Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 20h<br>18h | fabric, cleanup, batten |

### Layer I (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 17h<br>11h | docs |

### Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Memory guard fails a gate by killing cmd/lym tier1 when the user slice is full from elsewhere** | [`memory-guard-infra-stop`](design-memory-guard-infra-stop.md) | darn |  | 0m<br>0m | gate |

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

### Bugs (8)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **A rejected push can force origin past a commit made by hand on GitHub** | [`push-over-github-hand-commit`](design-push-over-github-hand-commit.md) | T23 | 18m<br>18m | fabric, landing |
| B2 |  | **A computed or base gate scope skips the guard tests of an impacted package whose tier it does not run** | [`scope-skips-impacted-guards`](design-scope-skips-impacted-guards.md) |  | 18m<br>18m | gate, testkit |
| B4 |  | **Finalize blames the fixer session for a push of origin/<task branch> it did not make** | [`finalize-fixer-halt-blame`](design-finalize-fixer-halt-blame.md) |  | 0m<br>0m | landing |
| B5 |  | **Finalize records a DNS failure classed transient as a failed / producer-hard-failure halt** | [`transient-stop-not-failed`](design-transient-stop-not-failed.md) |  | 0m<br>0m | shed, loom |
| B6 |  | **Test-only-code guard fails a card whose function gets its first caller in a later card** | [`plan-later-caller-guard`](design-plan-later-caller-guard.md) |  | 0m<br>0m | webster, loom |
| B7 |  | **Command-help byte-cap test is not run by card or batch gates** | [`cli-form-test-in-card-gate`](design-cli-form-test-in-card-gate.md) |  | 0m<br>0m | gate, cli |
| B8 |  | **Foreground lym gate test is backgrounded after 180 s despite a one-hour Bash timeout** | [`gate-call-backgrounded-180s`](design-gate-call-backgrounded-180s.md) |  | 0m<br>0m | shuttle, gate |
| B9 |  | **shell-edit guard passes perl -pi / sed -i on a line after a heredoc** | [`shellguard-heredoc-edit`](design-shellguard-heredoc-edit.md) |  | 0m<br>0m | process |

### Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Slow whole-repo guards, .NET builds and testmap tests leave the task gates for landing or periodic upkeep** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) |  | 10h<br>19m | gate, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) |  | 18h<br>18h | slack, creel |
| E3 | H | **Finalize gates the merged tree without the parent lock and holds the lock only to land** | [`landing-lock-only-for-land`](design-landing-lock-only-for-land.md) |  | 48m<br>48m | landing, gate |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | E3 | 48m<br>48m | landing, gate, batten |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs** | [`kde-window-placement`](design-kde-window-placement.md) | 10h<br>10h | termwindow, desktop |
