# P0 Root: entries under no project

Rendered 2026-11-27 16:06 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 1h<br>1h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T4 | **PR-Review in review mode stalls with no agent session** | [`pr-review-stalls-no-session`](design-pr-review-stalls-no-session.md) | darn · light | Code-Burler-Review | neve | 46m<br>45m | landing, loom, shuttle |
| T8 | **Re-step prompt after an interrupted run reports prior work as (none) though the branch carries the earlier spawn's commits** | [`restep-prior-work-none`](design-restep-prior-work-none.md) | darn · light | Code-Burler-Fix | neve | 45m<br>45m | loom, stencil |
| T9 | **twill nested-spawn check counts a parent's spawns inherited into a fork's transcript, halting the run** | [`twill-nested-spawn-inherited`](design-twill-nested-spawn-inherited.md) | darn · light | Code-Burler-Fix | neve | 45m<br>45m | burler |
| T13 | **Webster run-exit audit flags a resumed Master earlier friction note** | [`webster-audit-old-friction-note`](design-webster-audit-old-friction-note.md) | darn · light | Darn-Write | neve | 2h<br>39m | webster |
| T18 | **Upkeep gate call is refused because gate.gates.upkeep lists no tier covering tier1** | [`upkeep-gate-lacks-tier1`](design-upkeep-gate-lacks-tier1.md) | darn · light | starting · Open-IDE | neve | 1h<br>39m | gate, config |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | T11 | 45m<br>45m | batten, fabric |

## Layer G (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11<br>T28 | 1d<br>6h | fabric, cleanup, batten |

## Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 1d<br>19h | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | 9h<br>1h | landing, gate, batten |
