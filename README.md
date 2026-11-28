# Board

Rendered 2026-11-28 13:42 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `on` · PR setting `off`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages](project-test-speed.md) (0)

[`test-speed`](design-test-speed.md) · Age 22h · Owner `neve`

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 11d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 6w<br>1d | burler, shed, orch, quarry, report |

## [P3 Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack](project-self-healing-autonomy.md) (5)

[`self-healing-autonomy`](design-self-healing-autonomy.md) · Age 4h · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T21 | **beam.autonomy takes false, true or unattended, and every reader reads the level** | [`autonomy-unattended-level`](design-autonomy-unattended-level.md)<br>[Sketch](decision-autonomy-unattended-level.md) | darn · solid | Darn-Write | neve | 4h<br>3h | dobby, config, cwd, board, orch, slack, loom |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **The settlement table of stop classes, loom verbs that wait for the driver to park, and the driver's halt count** | [`stop-settlement-table`](design-stop-settlement-table.md)<br>[Sketch](decision-stop-settlement-table.md) | darn · solid | T21 | 4h<br>3h | shed, loom, batten |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T33 | **The driver and the arbiter settle known stops and budget escalations themselves within the table's bound** | [`driver-settles-stops`](design-driver-settles-stops.md)<br>[Sketch](decision-driver-settles-stops.md) | darn · solid | T12 | 4h<br>3h | shed, board, batten, shuttle, webster, arbiter, loom, stencil |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T32 | **The dobby resumes lifted pauses, posts severe halts to Slack, keeps quiet on correct waits and chases unsettled stalls** | [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)<br>[Sketch](decision-dobby-pause-lift-resume.md) | darn · solid | T33<br>T2 | 4h<br>3h | dobby, tender, batten, slack, creel, shuttle, loom, config |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T38 | **The orch role texts gain the unattended level's rules and the glossary names the level** | [`orch-unattended-rules`](design-orch-unattended-rules.md)<br>[Sketch](decision-orch-unattended-rules.md) | darn · docs | T33<br>T32 | 4h<br>3h | orch, heddle |

## [P4 Fabric split into a git-free core, a layout package and a thin git engine](project-fabric-core.md) (6)

[`fabric-core`](design-fabric-core.md) · Age 1d · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T23 | **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | Plan-Burler-Review | neve | 1d<br>3h | fabric, testkit, shed, board, pattern |

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

## [P5 Tapestry chains plan the next task while the one before still builds](project-plan-ahead-chains.md) (7)

[`plan-ahead-chains`](design-plan-ahead-chains.md) · Age 1h · Owner `neve`

### Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **Jacquard recipe holds tapestry's rows after Discussion, tapestry becomes the Motif and Jacquard blocks, and a Record-Seed row seeds the stored record** | [`jacquard-recipe`](design-jacquard-recipe.md)<br>[Sketch](decision-jacquard-recipe.md) | darn · solid | Darn-Write | neve | 5m<br>5m | shed, loom, testkit |
| T18 | **Every run profile with a tapestry block gains a jacquard block, carried over from the operator's tapestry values once at reconcile** | [`jacquard-run-profiles`](design-jacquard-run-profiles.md)<br>[Sketch](decision-jacquard-run-profiles.md) | darn · light | Darn-Write | neve | 5m<br>5m | profiles, config |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Batten, loom verbs and the report admit jacquard where it walks tapestry's path, and cost cells count a discussed tapestry run as jacquard** | [`jacquard-recipe-lists`](design-jacquard-recipe-lists.md)<br>[Sketch](decision-jacquard-recipe-lists.md) | darn · light | T9 | 5m<br>5m | batten, loom, report, tools |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Jacquard refuses an entry with no stored record at batten's start and at a first lym loom start, and the decision verbs name the seed row** | [`jacquard-start-refusal`](design-jacquard-start-refusal.md)<br>[Sketch](decision-jacquard-start-refusal.md) | darn · light | T11 | 5m<br>5m | loom, batten |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Raddle names jacquard for the parts it files, and the glossary and docs name jacquard and tapestry's two blocks** | [`raddle-names-jacquard`](design-raddle-names-jacquard.md)<br>[Sketch](decision-raddle-names-jacquard.md) | darn · light | T13<br>T18 | 5m<br>5m | parsers, loom, stencil, heddle, docs |

### Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F2 |  | **A dependent task plans ahead from its dependency's record and plan, and waits only to build** | [`pipelined-chain-planning`](design-pipelined-chain-planning.md) | T19<br>F3 | 2h<br>5m | loom, batten, shed |
| F3 |  | **Released tasks start in a queue order the board shows and the operator can change** | [`release-order-queue`](design-release-order-queue.md) |  | 1h<br>59m | board, dobby |

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

## [P8 Webster and Weaver spend less of their row in gate runs](project-implementer-gate-time.md) (5)

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 1h · Owner `neve`

### Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **A gate run under the parent lock takes the next free gate slot ahead of runs that hold no lock** | [`lock-holder-gates-first`](design-lock-holder-gates-first.md) | darn · speed | landing · Finalize | neve | 1h<br>43m | gate, landing |
| T6 | **The batch gate runs tier1 over the batch own packages, not their dependents** | [`batch-gate-tier1-own-packages`](design-batch-gate-tier1-own-packages.md) | darn · light | Darn-Write | neve | 17m<br>17m | gate, webster |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 |  | **The card gate leaves the guard step to the batch gate** | [`card-gate-no-guard-step`](design-card-gate-no-guard-step.md) | 1h<br>20m | gate, webster, weaver |
| E6 |  | **An implementer batch gets the files and symbols its cards name, so it starts editing sooner** | [`implementer-orientation-read`](design-implementer-orientation-read.md) | 1h<br>20m | webster, weaver, stencil |
| E7 |  | **The last batch gate covers cmd/lym and cross-package tests, so the plan-level verify seldom fails** | [`batch-gate-covers-cmd-lym`](design-batch-gate-covers-cmd-lym.md) | 1h<br>20m | gate, webster, weaver |

## [P0 Root: entries under no project](project-root.md) (18)

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

### Running (4)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Weaver gets a batch timeout of its own, names deviations in its summary, records gate spans and is tested on the paths its bench missed** | [`weaver-bench-fixes`](design-weaver-bench-fixes.md) | darn · solid | landing · Finalize | neve | 2h<br>2h | weaver, shed, shuttle |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Parent-Align | neve | 23h<br>23h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T10 | **Darn measurement rule covers a run with no parent and bars subagents from measurement calls** | [`darn-measure-rule-gaps`](design-darn-measure-rule-gaps.md) | darn · light | landing · Finalize | neve | 1h<br>1h | darn, stencil |
| T30 | **lym fabric clone --reset removes unpushed records, board commits and uncommitted Bolt changes** | [`clone-reset-keeps-unpushed`](design-clone-reset-keeps-unpushed.md) | darn · light | Code-Burler-Fix | neve | 3h<br>3h | fabric |

### Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips** | [`landing-ledger-own-tip`](design-landing-ledger-own-tip.md)<br>[Sketch](decision-landing-ledger-own-tip.md) | darn · light |  | 4m<br>4m | loom, landing |

### Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | T1 | 2h<br>2h | weaver, webster |
| T20 | **Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run** | [`landing-queue-batch`](design-landing-queue-batch.md)<br>[Sketch](decision-landing-queue-batch.md) | tapestry · solid | T3<br>T23 | 4m<br>4m | landing, fabric, git, cwd, loom, shed, docs, pattern, heddle |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2 | 5h<br>4h | termwindow, desktop |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure how much landing-lock-only-for-land cut the wait for the parent lock** | [`landing-lock-effect-measure`](design-landing-lock-effect-measure.md) | 4h<br>2h | landing, batten |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

### Bugs (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Webster run-exit audit flags Master friction note as a parent write** | [`webster-audit-master-friction`](design-webster-audit-master-friction.md) |  | 8m<br>8m | webster |
| B2 |  | **A window whose home desktop no longer exists keeps that home after it is placed elsewhere** | [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md) | T2 | 8m<br>8m | termwindow |

### Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) |  | 3h<br>3h | testkit, landing |
| E2 |  | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) |  | 3h<br>3h | testkit, creel, reed |
| E3 |  | **The PR-Review mail test takes its CommitRecord, BaseCommand and GateTiers seams from loomcli wiring instead of envkit stand-ins** | [`pr-mail-test-wired-seams`](design-pr-mail-test-wired-seams.md) |  | 2h<br>20m | testkit, landing |
| E5 |  | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 1h<br>1h | weaver, shed |

### Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 16h<br>3h | board, orch, dobby |
| F4 |  | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 1h<br>1h | shed, weaver |
