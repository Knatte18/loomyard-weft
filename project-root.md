# P0 Root: entries under no project

Rendered 2026-11-28 10:24 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (7)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **A missing local origin is not classed as unreachable by push or origin reconcile** | [`missing-origin-unreachable`](design-missing-origin-unreachable.md) | darn · light | Code-Burler-Review | neve | 12h<br>12h | fabric, git |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 19h<br>19h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | Code-Burler-Review | neve | 1d<br>1d | docs |
| T4 | **Finalize: a parent ahead by owed landing tips loses every race and verifies under the parent lock** | [`finalize-owed-tips-gate`](design-finalize-owed-tips-gate.md) | darn · light | landing · Finalize | neve | 12h<br>12h | landing |
| T5 | **PR-Review session tests run over a real creel daemon and the recorded tmux kit, which learns to replay pastes**<br>• Move the MailAgent rows to a tier1 test over a real creel daemon and drop the envkit stand-ins | [`pr-session-mail-stand-ins`](design-pr-session-mail-stand-ins.md) | darn · solid | Code-Burler-Fix | neve | 15h<br>1h | testkit, landing, reed, creel |
| T8 | **runpusher tries to start pushers for removed task worktrees** | [`runpusher-removed-worktrees`](design-runpusher-removed-worktrees.md) | darn · light | Darn-Write | neve | 12h<br>12h | fabric |
| T10 | **A catch-up fetch that fails on an unreachable origin still stops a run start**<br>• push-retry-not-halt made pushes retry quietly, but dobby catch-up fetch failure still refuses the start | [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md) | darn · light | Code-Burler-Fix | neve | 12h<br>1h | dobby, fabric |

## Ready (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Discussion-Bouncer reports stalemate when the fixer's decision-record edits are uncommitted** | [`bouncer-stalemate-uncommitted`](design-bouncer-stalemate-uncommitted.md) | darn · light |  | 12h<br>12h | shed, burler |
| T13 | **Reviews and plans check their claims against the code: fix keys, test infrastructure, card claims and first callers** | [`review-fix-claims-checked`](design-review-fix-claims-checked.md) | darn · light |  | 12h<br>12h | burler, stencil, webster |
| T18 | **Darn writer prompt forbids the gate calls a measurement entry requires** | [`darn-writer-measure-gates`](design-darn-writer-measure-gates.md) | darn · light |  | 12h<br>12h | darn, stencil |
| T29 | **A running beam orch reads as down, and a strand missing from reed state is not covered** | [`beam-orch-reads-down`](design-beam-orch-reads-down.md) | darn · light |  | 36m<br>36m | orch, reed, cwd |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T22 | **A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted** | [`window-retile-verb`](design-window-retile-verb.md) | darn · light | T2 | 1h<br>1h | termwindow, desktop |
| T30 | **lym fabric clone --reset removes unpushed records, board commits and uncommitted Bolt changes** | [`clone-reset-keeps-unpushed`](design-clone-reset-keeps-unpushed.md) | darn · light | T1<br>T8<br>T10 | 36m<br>36m | fabric |

## Research (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure how much landing-lock-only-for-land cut the wait for the parent lock** | [`landing-lock-effect-measure`](design-landing-lock-effect-measure.md) | T4 | 58m<br>58m | landing, batten |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) |  | 10d<br>10d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E4 | L | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | R1 | 1d<br>58m | landing, gate, batten |

## Features (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | T38 | 12h<br>13m | board, orch, dobby |
