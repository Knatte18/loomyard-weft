# P0 Root: entries under no project

Rendered 2026-11-28 15:36 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Code-Burler-Fix | neve | 1d<br>1d | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | Darn-Write | neve | 4h<br>4h | weaver, webster |
| T10 | **Test git can reach the network and open a credential prompt, hanging an agent's test run** | [`test-git-hermetic-network`](design-test-git-hermetic-network.md) | darn · light | Darn-Write | neve | 19m<br>18m | testkit, git |
| T18 | **Review fork guard halts the run on a push from the fork's own throwaway repo after an ln, with a message that names the wrong rule** | [`review-fork-guard-ln-push`](design-review-fork-guard-ln-push.md) | darn · light | Darn-Write | neve | 19m<br>18m | burler, stencil |
| T20 | **Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run** | [`landing-queue-batch`](design-landing-queue-batch.md)<br>[Sketch](decision-landing-queue-batch.md) | tapestry · solid | starting · Seed-Child | neve | 1h<br>15m | landing, fabric, git, cwd, loom, shed, docs, pattern, heddle |
| T31 | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) | darn · light | Darn-Write | neve | 5h<br>17m | testkit, creel, reed |

## Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T34 | **Webster run-exit audit flags Master friction note as a parent write** | [`webster-audit-master-friction`](design-webster-audit-master-friction.md) | darn · light |  | 2h<br>11m | webster |

## Layer A (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Gate tier timeout leaves out suspended time, and --gate with explicit targets runs or refuses marked tests** | [`gate-suspend-marked-targets`](design-gate-suspend-marked-targets.md) | darn · light | T1 | 19m<br>18m | gate, hibernate |
| T29 | **Parent-Align halts when its own fixer pushes the task branch, reading the fixer commits as a foreign move** | [`parent-align-fixer-own-push`](design-parent-align-fixer-own-push.md) | darn · light | T20 | 48m<br>15m | landing |
| T35 | **A window whose home desktop no longer exists keeps that home after it is placed elsewhere** | [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md) | darn · light | T2 | 2h<br>11m | termwindow |
| T36 | **lym fabric clone --reset refuses a hub holding leftovers of an earlier _bolt layout** | [`reset-old-layout-leftovers`](design-reset-old-layout-leftovers.md) | darn · light | T23 | 52m<br>11m | fabric |

## Layer B (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2<br>T35 | 6h<br>11m | termwindow, desktop |
| T30 | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) | darn · light | T29 | 5h<br>17m | testkit, landing |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B5 |  | **Dobby posts a refused beam.autonomy on its own tick** | [`autonomy-refusal-own-tick`](design-autonomy-refusal-own-tick.md) | 29m<br>29m | dobby |

## Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E5 | L | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 3h<br>18m | weaver, shed |

## Features (3)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 17h<br>5h | board, orch, dobby |
| F4 | L | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 3h<br>18m | shed, weaver |
| F5 |  | **Gate runs on a LAN machine: Septim takes gate slots through a lym service in WSL2** | [`remote-gate-farm`](design-remote-gate-farm.md) |  | 0m<br>0m | gate, process, hibernate |
