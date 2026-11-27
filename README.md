# Board

Rendered 2026-11-27 11:30 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `off` · PR setting `off`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 10d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 6w<br>3h | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (1)

`intake-wave-2` · Age 1d · Owner `neve`

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | tapestry | T6<br>T8 | 1d<br>13m | fabric, shed, landing, batten, dobby |

## [P4 Fabric split into a git-free core, a layout package and a thin git engine](project-fabric-core.md) (7)

[`fabric-core`](design-fabric-core.md) · Age 13h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Fabric core foundation: layout and core packages, snapshot, action list, executor loop and the layout retarget** | [`fabric-core-foundation`](design-fabric-core-foundation.md)<br>[Sketch](decision-fabric-core-foundation.md) | tapestry · solid | Webster | neve | 13h<br>13h | fabric, testkit, pattern, cwd, loom, batten, landing, gate, orch, board, webster, drift, report |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T23 | **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | T6<br>T11 | 13h<br>1h | fabric, testkit, shed, board, pattern |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T24 | **Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core** | [`fabric-core-teardown`](design-fabric-core-teardown.md)<br>[Sketch](decision-fabric-core-teardown.md) | tapestry · solid | T23 | 13h<br>13h | fabric, testkit, shed, landing |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T25 | **Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core** | [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)<br>[Sketch](decision-fabric-core-merge-landing.md) | tapestry · solid | T24 | 13h<br>13h | fabric, testkit, shed, batten, landing |

### Layer E (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T26 | **Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric** | [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)<br>[Sketch](decision-fabric-core-pull-replay.md) | tapestry · solid | T25 | 13h<br>13h | fabric, testkit, shed, stencil, board, heddle |

### Layer F (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T27 | **Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut** | [`fabric-core-add-clone`](design-fabric-core-add-clone.md)<br>[Sketch](decision-fabric-core-add-clone.md) | tapestry · solid | T26 | 13h<br>13h | fabric, testkit, ide, batten, heddle, reconcile, shed |

### Layer G (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T28 | **Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure** | [`fabric-core-records-guards`](design-fabric-core-records-guards.md)<br>[Sketch](decision-fabric-core-records-guards.md) | tapestry · solid | T27 | 13h<br>13h | fabric, testkit, pattern, docs |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (1)

`intake-wave-3` · Age 1d · Owner `neve`

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | T11 | 1d<br>22h | batten, landing, orch, stencil, shed, loom, pawl |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (1)

`intake-motifs` · Age 1d · Owner `neve`

### Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T11<br>T12<br>T28 | 23h<br>1h | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 23h · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 23h<br>23h | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T11 | 23h<br>23h | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 23h<br>23h | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 23h<br>23h | termwindow, hibernate, batten, ide, deploy, reed |

## [P0 Root: entries under no project](project-root.md) (9)

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

### Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Slow whole-repo guards and .NET builds leave the task gates for landing or periodic upkeep** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) | darn · solid | Darn-Write | neve | 14h<br>1h | gate, testkit |
| T8 | **Finalize gates the merged tree without the parent lock and holds the lock only to land** | [`landing-lock-only-for-land`](design-landing-lock-only-for-land.md) | darn · solid | Darn-Write | neve | 4h<br>1h | landing, gate |
| T10 | **A bench run creates and removes its own bench pairs, forked from its task worktree** | [`bench-makes-own-pairs`](design-bench-makes-own-pairs.md) | darn · solid | Code-Bouncer | neve | 58m<br>55m | fabric, darn, batten |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **Bench Weaver against Webster on plans Webster already implemented: time, cost and quality** | [`weaver-bench`](design-weaver-bench.md) | darn · docs | T10 | 1h<br>55m | weaver, webster |

### Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11<br>T28 | 1d<br>1h | fabric, cleanup, batten |

### Layer I (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 21h<br>15h | docs |

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs** | [`kde-window-placement`](design-kde-window-placement.md)<br>[Sketch](decision-kde-window-placement.md) | 14h<br>1m | termwindow, desktop |

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

### Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | T8 | 4h<br>4h | landing, gate, batten |
