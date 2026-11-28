# P0 Root: entries under no project

Rendered 2026-11-28 13:30 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Weaver gets a batch timeout of its own, names deviations in its summary, records gate spans and is tested on the paths its bench missed** | [`weaver-bench-fixes`](design-weaver-bench-fixes.md) | darn · solid | landing · Finalize | neve | 1h<br>1h | weaver, shed, shuttle |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 22h<br>22h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T8 | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md)<br>[Sketch](decision-landing-queue.md) | raddle | Raddle-Write | neve | 1d<br>1m | landing, gate, batten |
| T10 | **Darn measurement rule covers a run with no parent and bars subagents from measurement calls** | [`darn-measure-rule-gaps`](design-darn-measure-rule-gaps.md) | darn · light | Code-Bouncer | neve | 51m<br>51m | darn, stencil |
| T30 | **lym fabric clone --reset removes unpushed records, board commits and uncommitted Bolt changes** | [`clone-reset-keeps-unpushed`](design-clone-reset-keeps-unpushed.md) | darn · light | Darn-Write | neve | 3h<br>3h | fabric |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Bench Weaver against Webster again on the same three landed plans, both run side by side the same day** | [`weaver-bench-rerun`](design-weaver-bench-rerun.md) | darn · docs | T1 | 1h<br>1h | weaver, webster |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2 | 4h<br>4h | termwindow, desktop |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure how much landing-lock-only-for-land cut the wait for the parent lock** | [`landing-lock-effect-measure`](design-landing-lock-effect-measure.md) | 4h<br>2h | landing, batten |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API** | [`pr-review-github-recorded`](design-pr-review-github-recorded.md) |  | 2h<br>2h | testkit, landing |
| E2 |  | **A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery** | [`creel-delivery-reed-tier1`](design-creel-delivery-reed-tier1.md) |  | 2h<br>2h | testkit, creel, reed |
| E3 |  | **The PR-Review mail test takes its CommitRecord, BaseCommand and GateTiers seams from loomcli wiring instead of envkit stand-ins** | [`pr-mail-test-wired-seams`](design-pr-mail-test-wired-seams.md) |  | 1h<br>8m | testkit, landing |
| E5 |  | **Weaver runs each batch as a Shed row of the new kind and drops its own stop, re-entry, resume and lock code** | [`weaver-onto-shed-rows`](design-weaver-onto-shed-rows.md) | F4 | 1h<br>1h | weaver, shed |

## Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 15h<br>3h | board, orch, dobby |
| F4 |  | **Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch** | [`shed-dynamic-rows`](design-shed-dynamic-rows.md) |  | 1h<br>1h | shed, weaver |
