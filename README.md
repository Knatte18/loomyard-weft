# Board

Rendered 2026-11-28 15:25 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `on` · PR setting `off`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages](project-test-speed.md) (1)

[`test-speed`](design-test-speed.md) · Age 1d · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Measure the test-speed cuts side by side: the packages they changed timed before and after under the same load** | [`test-speed-effect-measure`](design-test-speed-effect-measure.md) | darn · docs | Darn-Write | neve | 38m<br>38m | gate, testkit |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 11d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 6w<br>1d | burler, shed, orch, quarry, report |

## [P3 Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack](project-self-healing-autonomy.md) (5)

[`self-healing-autonomy`](design-self-healing-autonomy.md) · Age 5h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T21 | **beam.autonomy takes false, true or unattended, and every reader reads the level** | [`autonomy-unattended-level`](design-autonomy-unattended-level.md)<br>[Sketch](decision-autonomy-unattended-level.md) | darn · solid | Code-Burler-Fix | neve | 5h<br>5h | dobby, config, cwd, board, orch, slack, loom |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **The settlement table of stop classes, loom verbs that wait for the driver to park, and the driver's halt count** | [`stop-settlement-table`](design-stop-settlement-table.md)<br>[Sketch](decision-stop-settlement-table.md) | darn · solid | T21 | 5h<br>5h | shed, loom, batten |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T33 | **The driver and the arbiter settle known stops and budget escalations themselves within the table's bound** | [`driver-settles-stops`](design-driver-settles-stops.md)<br>[Sketch](decision-driver-settles-stops.md) | darn · solid | T12 | 5h<br>5h | shed, board, batten, shuttle, webster, arbiter, loom, stencil |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T32 | **The dobby resumes lifted pauses, posts severe halts to Slack, keeps quiet on correct waits and chases unsettled stalls** | [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)<br>[Sketch](decision-dobby-pause-lift-resume.md) | darn · solid | T33<br>T2 | 5h<br>5h | dobby, tender, batten, slack, creel, shuttle, loom, config |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T38 | **The orch role texts gain the unattended level's rules and the glossary names the level** | [`orch-unattended-rules`](design-orch-unattended-rules.md)<br>[Sketch](decision-orch-unattended-rules.md) | darn · docs | T33<br>T32 | 5h<br>5h | orch, heddle |

## [P4 Fabric split into a git-free core, a layout package and a thin git engine](project-fabric-core.md) (6)

[`fabric-core`](design-fabric-core.md) · Age 1d · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T23 | **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | Plan-Burler-Fix | neve | 1d<br>4h | fabric, testkit, shed, board, pattern |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T24 | **Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core** | [`fabric-core-teardown`](design-fabric-core-teardown.md)<br>[Sketch](decision-fabric-core-teardown.md) | tapestry · solid | T23 | 1d<br>1d | fabric, testkit, shed, landing |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T25 | **Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core** | [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)<br>[Sketch](decision-fabric-core-merge-landing.md) | tapestry · solid | T24 | 1d<br>1d | fabric, testkit, shed, batten, landing |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T26 | **Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric** | [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)<br>[Sketch](decision-fabric-core-pull-replay.md) | tapestry · solid | T25 | 1d<br>1d | fabric, testkit, shed, stencil, board, heddle |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T27 | **Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut** | [`fabric-core-add-clone`](design-fabric-core-add-clone.md)<br>[Sketch](decision-fabric-core-add-clone.md) | tapestry · solid | T26 | 1d<br>1d | fabric, testkit, ide, batten, heddle, reconcile, shed |

### Layer E (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T28 | **Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure** | [`fabric-core-records-guards`](design-fabric-core-records-guards.md)<br>[Sketch](decision-fabric-core-records-guards.md) | tapestry · solid | T27 | 1d<br>1d | fabric, testkit, pattern, docs |

## [P5 Tapestry chains plan the next task while the one before still builds](project-plan-ahead-chains.md) (6)

[`plan-ahead-chains`](design-plan-ahead-chains.md) · Age 3h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **Jacquard recipe holds tapestry's rows after Discussion, tapestry becomes the Motif and Jacquard blocks, and a Record-Seed row seeds the stored record** | [`jacquard-recipe`](design-jacquard-recipe.md)<br>[Sketch](decision-jacquard-recipe.md) | darn · solid | Code-Bouncer | neve | 1h<br>1h | shed, loom, testkit |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Batten, loom verbs and the report admit jacquard where it walks tapestry's path, and cost cells count a discussed tapestry run as jacquard** | [`jacquard-recipe-lists`](design-jacquard-recipe-lists.md)<br>[Sketch](decision-jacquard-recipe-lists.md) | darn · light | T9 | 1h<br>1h | batten, loom, report, tools |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Jacquard refuses an entry with no stored record at batten's start and at a first lym loom start, and the decision verbs name the seed row** | [`jacquard-start-refusal`](design-jacquard-start-refusal.md)<br>[Sketch](decision-jacquard-start-refusal.md) | darn · light | T11 | 1h<br>1h | loom, batten |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Raddle names jacquard for the parts it files, and the glossary and docs name jacquard and tapestry's two blocks** | [`raddle-names-jacquard`](design-raddle-names-jacquard.md)<br>[Sketch](decision-raddle-names-jacquard.md) | darn · light | T13 | 1h<br>1h | parsers, loom, stencil, heddle, docs |

### Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F2 |  | **A dependent task plans ahead from its dependency's record and plan, and waits only to build** | [`pipelined-chain-planning`](design-pipelined-chain-planning.md) | T19<br>F3 | 4h<br>1h | loom, batten, shed |
| F3 |  | **Released tasks start in a queue order the board shows and the operator can change** | [`release-order-queue`](design-release-order-queue.md) |  | 2h<br>2h | board, dobby |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (1)

`intake-motifs` · Age 2d · Owner `neve`

### Layer F (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T28 | 2d<br>1d | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 2d · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 2d<br>2d | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7 | 2d<br>2d | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 2d<br>2d | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 2d<br>2d | termwindow, hibernate, batten, ide, deploy, reed |

## [P8 Webster and Weaver spend less of their row in gate runs](project-implementer-gate-time.md) (3)

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 3h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **The batch gate runs tier1 over the batch own packages, not their dependents** | [`batch-gate-tier1-own-packages`](design-batch-gate-tier1-own-packages.md) | darn · light | Friction-Reflect | neve | 2h<br>2h | gate, webster |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **The card gate runs over the card own packages and its fixer stops at the failed check** | [`card-gate-own-packages`](design-card-gate-own-packages.md) | darn · light | T6 | 3h<br>54m | gate, webster, weaver |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E6 |  | **An implementer batch gets the files and symbols its cards name, so it starts editing sooner** | [`implementer-orientation-read`](design-implementer-orientation-read.md) | 3h<br>2h | webster, weaver, stencil |

## [P0 Root: entries under no project](project-root.md) (19)

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

### Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Code-Burler-Fix | neve | 1d<br>1d | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T3 | **A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips** | [`landing-ledger-own-tip`](design-landing-ledger-own-tip.md)<br>[Sketch](decision-landing-ledger-own-tip.md) | darn · light | landing · Finalize | neve | 1h<br>1h | loom, landing |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | Darn-Write | neve | 3h<br>3h | weaver, webster |
| T10 | **Test git can reach the network and open a credential prompt, hanging an agent's test run** | [`test-git-hermetic-network`](design-test-git-hermetic-network.md) | darn · light | Darn-Write | neve | 8m<br>7m | testkit, git |
| T18 | **Review fork guard halts the run on a push from the fork's own throwaway repo after an ln, with a message that names the wrong rule** | [`review-fork-guard-ln-push`](design-review-fork-guard-ln-push.md) | darn · light | Darn-Write | neve | 8m<br>7m | burler, stencil |

### Ready (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T31 | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) | darn · light |  | 4h<br>6m | testkit, creel, reed |
| T34 | **Webster run-exit audit flags Master friction note as a parent write** | [`webster-audit-master-friction`](design-webster-audit-master-friction.md) | darn · light |  | 1h<br>0m | webster |

### Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T20 | **Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run** | [`landing-queue-batch`](design-landing-queue-batch.md)<br>[Sketch](decision-landing-queue-batch.md) | tapestry · solid | T3 | 1h<br>4m | landing, fabric, git, cwd, loom, shed, docs, pattern, heddle |
| T35 | **A window whose home desktop no longer exists keeps that home after it is placed elsewhere** | [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md) | darn · light | T2 | 1h<br>0m | termwindow |
| T36 | **lym fabric clone --reset refuses a hub holding leftovers of an earlier _bolt layout** | [`reset-old-layout-leftovers`](design-reset-old-layout-leftovers.md) | darn · light | T23 | 41m<br>0m | fabric |

### Layer B (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Gate tier timeout leaves out suspended time, and --gate with explicit targets runs or refuses marked tests** | [`gate-suspend-marked-targets`](design-gate-suspend-marked-targets.md) | darn · light | T1 | 8m<br>7m | gate, hibernate |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2<br>T35 | 6h<br>0m | termwindow, desktop |
| T29 | **Parent-Align halts when its own fixer pushes the task branch, reading the fixer commits as a foreign move** | [`parent-align-fixer-own-push`](design-parent-align-fixer-own-push.md) | darn · light | T3<br>T20 | 37m<br>4m | landing |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T30 | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) | darn · light | T29 | 4h<br>6m | testkit, landing |

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B5 |  | **Dobby posts a refused beam.autonomy on its own tick** | [`autonomy-refusal-own-tick`](design-autonomy-refusal-own-tick.md) | 18m<br>18m | dobby |

### Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E5 | L | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 3h<br>7m | weaver, shed |

### Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 17h<br>5h | board, orch, dobby |
| F4 | L | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 3h<br>7m | shed, weaver |
