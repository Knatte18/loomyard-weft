# P0 Root: entries under no project

Rendered 2026-11-27 10:30 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs** | [`kde-window-placement`](design-kde-window-placement.md) | motif | Discussion-Burler-Review | neve | 13h<br>20m | termwindow, desktop |
| T2 | **Finalize blames the fixer session for a push of origin/<task branch> it did not make** | [`finalize-fixer-halt-blame`](design-finalize-fixer-halt-blame.md) | darn · light | Describe | neve | 2h<br>43m | landing |
| T19 | **Remove the coverage maps and every user of them, keeping the redundancy sweep and red-first** | [`remove-testmap`](design-remove-testmap.md) | darn · solid | Code-Burler-Fix | neve | 2h<br>2h | testmap, gate, darn, loom, stencil, cleanup |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Finalize records a DNS failure classed transient as a failed / producer-hard-failure halt** | [`transient-stop-not-failed`](design-transient-stop-not-failed.md) | darn · light | T2 | 2h<br>2h | shed, loom |
| T5 | **Slow whole-repo guards and .NET builds leave the task gates for landing or periodic upkeep** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) | darn · solid | T19 | 13h<br>4m | gate, testkit |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Finalize gates the merged tree without the parent lock and holds the lock only to land** | [`landing-lock-only-for-land`](design-landing-lock-only-for-land.md) | darn · solid | T4 | 3h<br>15m | landing, gate |

## Layer J (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11<br>T28 | 23h<br>47m | fabric, cleanup, batten |

## Layer K (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 20h<br>14h | docs |

## Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **Bench Weaver against Webster on plans Webster already implemented: time, cost and quality** | [`weaver-bench`](design-weaver-bench.md) | darn · docs |  | 2m<br>0m | weaver, webster |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | T8 | 3h<br>3h | landing, gate, batten |
