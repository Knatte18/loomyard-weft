# P0 Root: entries under no project

Rendered 2026-11-28 15:05 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Code-Burler-Review | neve | 1d<br>1d | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T3 | **A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips** | [`landing-ledger-own-tip`](design-landing-ledger-own-tip.md)<br>[Sketch](decision-landing-ledger-own-tip.md) | darn · light | landing · Finalize | neve | 1h<br>1h | loom, landing |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | Darn-Write | neve | 3h<br>3h | weaver, webster |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T20 | **Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run** | [`landing-queue-batch`](design-landing-queue-batch.md)<br>[Sketch](decision-landing-queue-batch.md) | tapestry · solid | T3<br>T23 | 1h<br>1h | landing, fabric, git, cwd, loom, shed, docs, pattern, heddle |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2 | 6h<br>6h | termwindow, desktop |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Webster run-exit audit flags Master friction note as a parent write** | [`webster-audit-master-friction`](design-webster-audit-master-friction.md) |  | 1h<br>1h | webster |
| B2 |  | **A window whose home desktop no longer exists keeps that home after it is placed elsewhere** | [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md) | T2 | 1h<br>1h | termwindow |
| B3 |  | **lym fabric clone --reset refuses a hub holding leftovers of an earlier _bolt layout** | [`reset-old-layout-leftovers`](design-reset-old-layout-leftovers.md) |  | 21m<br>21m | fabric |
| B4 |  | **Parent-Align halts when its own fixer pushes the task branch, reading the fixer commits as a foreign move** | [`parent-align-fixer-own-push`](design-parent-align-fixer-own-push.md) |  | 16m<br>16m | landing |

## Enhancements (3)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) |  | 4h<br>4h | testkit, landing |
| E2 |  | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) |  | 4h<br>4h | testkit, creel, reed |
| E5 |  | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 3h<br>3h | weaver, shed |

## Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 17h<br>4h | board, orch, dobby |
| F4 |  | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 3h<br>3h | shed, weaver |
