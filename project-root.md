# P0 Root: entries under no project

Rendered 2026-11-28 13:56 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (4)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Parent-Align | neve | 23h<br>23h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | Darn-Write | neve | 2h<br>2h | weaver, webster |
| T10 | **Darn measurement rule covers a run with no parent and bars subagents from measurement calls** | [`darn-measure-rule-gaps`](design-darn-measure-rule-gaps.md) | darn · light | landing · Finalize | neve | 1h<br>1h | darn, stencil |
| T30 | **lym fabric clone --reset removes unpushed records, board commits and uncommitted Bolt changes** | [`clone-reset-keeps-unpushed`](design-clone-reset-keeps-unpushed.md) | darn · light | Code-Burler-Fix | neve | 4h<br>4h | fabric |

## Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips** | [`landing-ledger-own-tip`](design-landing-ledger-own-tip.md)<br>[Sketch](decision-landing-ledger-own-tip.md) | darn · light |  | 18m<br>18m | loom, landing |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T20 | **Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run** | [`landing-queue-batch`](design-landing-queue-batch.md)<br>[Sketch](decision-landing-queue-batch.md) | tapestry · solid | T3<br>T23 | 18m<br>18m | landing, fabric, git, cwd, loom, shed, docs, pattern, heddle |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2 | 5h<br>5h | termwindow, desktop |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure how much landing-lock-only-for-land cut the wait for the parent lock** | [`landing-lock-effect-measure`](design-landing-lock-effect-measure.md) | 4h<br>2h | landing, batten |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **Webster run-exit audit flags Master friction note as a parent write** | [`webster-audit-master-friction`](design-webster-audit-master-friction.md) |  | 22m<br>22m | webster |
| B2 |  | **A window whose home desktop no longer exists keeps that home after it is placed elsewhere** | [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md) | T2 | 22m<br>22m | termwindow |

## Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) |  | 3h<br>3h | testkit, landing |
| E2 |  | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) |  | 3h<br>3h | testkit, creel, reed |
| E3 |  | **The PR-Review mail test takes its CommitRecord, BaseCommand and GateTiers seams from loomcli wiring instead of envkit stand-ins** | [`pr-mail-test-wired-seams`](design-pr-mail-test-wired-seams.md) |  | 2h<br>34m | testkit, landing |
| E5 |  | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 2h<br>2h | weaver, shed |

## Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 16h<br>3h | board, orch, dobby |
| F4 |  | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 2h<br>2h | shed, weaver |
