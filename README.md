# Board

Rendered 2026-11-28 08:45 UTC; every age below is as of then.
Prime owner: `neve`.

Hub-wide default: autonomy `on` · PR setting `off`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Faster gates: measure where test time goes, then cut or restructure the heavy packages](project-test-speed.md) (1)

[`test-speed`](design-test-speed.md) · Age 17h · Owner `neve`

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E6 |  | **TestGateRedFirst_Dotnet sets the floor of the gatecli tier1 time at 31 to 47 s**<br>• After the scenario split, one .NET red-first test holds gatecli tier1 near its old time | [`gatecli-dotnet-redfirst-time`](design-gatecli-dotnet-redfirst-time.md) | 14h<br>13h | gate |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 10d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 6w<br>1d | burler, shed, orch, quarry, report |

## [P4 Fabric split into a git-free core, a layout package and a thin git engine](project-fabric-core.md) (6)

[`fabric-core`](design-fabric-core.md) · Age 1d · Owner `neve`

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T23 | **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | T21<br>T1<br>T8 | 1d<br>11h | fabric, testkit, shed, board, pattern |

### Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T24 | **Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core** | [`fabric-core-teardown`](design-fabric-core-teardown.md)<br>[Sketch](decision-fabric-core-teardown.md) | tapestry · solid | T23 | 1d<br>1d | fabric, testkit, shed, landing |

### Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T25 | **Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core** | [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)<br>[Sketch](decision-fabric-core-merge-landing.md) | tapestry · solid | T24 | 1d<br>1d | fabric, testkit, shed, batten, landing |

### Layer E (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T26 | **Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric** | [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)<br>[Sketch](decision-fabric-core-pull-replay.md) | tapestry · solid | T25 | 1d<br>1d | fabric, testkit, shed, stencil, board, heddle |

### Layer F (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T27 | **Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut** | [`fabric-core-add-clone`](design-fabric-core-add-clone.md)<br>[Sketch](decision-fabric-core-add-clone.md) | tapestry · solid | T26 | 1d<br>1d | fabric, testkit, ide, batten, heddle, reconcile, shed |

### Layer G (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T28 | **Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure** | [`fabric-core-records-guards`](design-fabric-core-records-guards.md)<br>[Sketch](decision-fabric-core-records-guards.md) | tapestry · solid | T27 | 1d<br>20h | fabric, testkit, pattern, docs |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (1)

`intake-wave-3` · Age 2d · Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T12 | **A beam member's run names its beam orch as parent, which may amend the beam's record** | [`beam-member-parent`](design-beam-member-parent.md) | darn · light | Code-Burler-Fix | neve | 2d<br>1d | batten, landing, orch, stencil, shed, loom, pawl |

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (1)

`intake-motifs` · Age 2d · Owner `neve`

### Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T12<br>T28 | 1d<br>22h | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 1d · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 1d<br>1d | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7 | 1d<br>1d | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 1d<br>1d | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 1d<br>1d | termwindow, hibernate, batten, ide, deploy, reed |

## [P0 Root: entries under no project](project-root.md) (21)

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

### Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 18h<br>18h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | landing · Finalize | neve | 17h<br>17h | batten, fabric |
| T19 | **Timing a gate test call includes the slot wait; run time needs the run record** | [`gate-call-shows-slot-wait`](design-gate-call-shows-slot-wait.md) | darn · light | landing · Finalize | neve | 11h<br>11h | gate |
| T20 | **webster reset refuses before its records push when fabricSync fails after the records commit landed** | [`reset-partial-commit-push`](design-reset-partial-commit-push.md) | darn · light | landing · Finalize | neve | 48m<br>48m | webster, fabric |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | Code-Burler-Fix | neve | 1d<br>14h | fabric, cleanup, batten |

### Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Arras still shows in seeded profiles and in the overview and glossary**<br>• The landed design-first route left arras keys in the profile template and descriptions in two docs | [`arras-visible-leftovers`](design-arras-visible-leftovers.md) | darn · light |  | 12h<br>0m | board, docs |

### Layer A (9)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **A missing local origin is not classed as unreachable by push or origin reconcile** | [`missing-origin-unreachable`](design-missing-origin-unreachable.md) | darn · light | T21 | 11h<br>11h | fabric, git |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T21 | 1d<br>1d | docs |
| T4 | **Finalize: a parent ahead by owed landing tips loses every race and verifies under the parent lock** | [`finalize-owed-tips-gate`](design-finalize-owed-tips-gate.md) | darn · light | T12 | 11h<br>11h | landing |
| T5 | **PR-Review session tests run over a real creel daemon and the recorded tmux kit, which learns to replay pastes**<br>• Move the MailAgent rows to a tier1 test over a real creel daemon and drop the envkit stand-ins | [`pr-session-mail-stand-ins`](design-pr-session-mail-stand-ins.md) | darn · solid | T12 | 13h<br>0m | testkit, landing, reed, creel |
| T8 | **runpusher tries to start pushers for removed task worktrees** | [`runpusher-removed-worktrees`](design-runpusher-removed-worktrees.md) | darn · light | T21 | 11h<br>11h | fabric |
| T9 | **Tier0 fails on a Go file gofmt would rewrite**<br>• A mis-formatted Go file passes every tier; add a gofmt check to tier0 | [`battencli-wire-test-gofmt`](design-battencli-wire-test-gofmt.md) | darn · light | T19 | 12h<br>0m | gate |
| T11 | **Discussion-Bouncer reports stalemate when the fixer's decision-record edits are uncommitted** | [`bouncer-stalemate-uncommitted`](design-bouncer-stalemate-uncommitted.md) | darn · light | T12 | 11h<br>11h | shed, burler |
| T13 | **Reviews and plans check their claims against the code: fix keys, test infrastructure, card claims and first callers** | [`review-fix-claims-checked`](design-review-fix-claims-checked.md) | darn · light | T12 | 11h<br>11h | burler, stencil, webster |
| T18 | **Darn writer prompt forbids the gate calls a measurement entry requires** | [`darn-writer-measure-gates`](design-darn-writer-measure-gates.md) | darn · light | T12 | 11h<br>11h | darn, stencil |

### Sketches (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| S1 |  | **Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack** | [`self-healing-runs`](design-self-healing-runs.md)<br>[Sketch](decision-self-healing-runs.md) | T10<br>T12 | 14h<br>11h | dobby, batten, orch |

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A catch-up fetch that fails on an unreachable origin still stops a run start**<br>• push-retry-not-halt made pushes retry quietly, but dobby catch-up fetch failure still refuses the start | [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md) | 11h<br>11h | dobby, fabric |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | L | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | 1d<br>14h | landing, gate, batten |

### Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) |  | 11h<br>11h | board, orch, dobby |
| F2 |  | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | T2 | 6m<br>6m | termwindow, desktop |
