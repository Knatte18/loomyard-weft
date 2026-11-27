# P0 Root: entries under no project

Rendered 2026-11-27 16:43 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 2h<br>2h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T4 | **PR-Review in review mode stalls with no agent session** | [`pr-review-stalls-no-session`](design-pr-review-stalls-no-session.md) | darn · light | landing · Finalize | neve | 1h<br>1h | landing, loom, shuttle |
| T8 | **Re-step prompt after an interrupted run reports prior work as (none) though the branch carries the earlier spawn's commits** | [`restep-prior-work-none`](design-restep-prior-work-none.md) | darn · light | landing · Finalize | neve | 1h<br>1h | loom, stencil |
| T13 | **Webster run-exit audit flags a resumed Master earlier friction note** | [`webster-audit-old-friction-note`](design-webster-audit-old-friction-note.md) | darn · light | landing · Finalize | neve | 3h<br>1h | webster |
| T18 | **Upkeep gate call is refused because gate.gates.upkeep lists no tier covering tier1** | [`upkeep-gate-lacks-tier1`](design-upkeep-gate-lacks-tier1.md) | darn · light | Code-Burler-Review | neve | 2h<br>1h | gate, config |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | T11 | 1h<br>1h | batten, fabric |

## Layer G (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11<br>T28 | 1d<br>7h | fabric, cleanup, batten |

## Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 1d<br>20h | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Reed agent button refuses a name with spaces without saying why** | [`reed-agent-name-spaces`](design-reed-agent-name-spaces.md) | 6m<br>6m | reed |

## Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests still hand-write reed, creel and session stand-ins** | [`pr-session-test-stand-ins`](design-pr-session-test-stand-ins.md) | 6m<br>6m | testkit, landing |
| E2 |  | **Agent spawned from the reed agent button starts with no lym context** | [`reed-agent-blank-context`](design-reed-agent-blank-context.md) | 6m<br>6m | reed, stencil |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | 9h<br>2h | landing, gate, batten |
