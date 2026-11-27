# P0 Root: entries under no project

Rendered 2026-11-27 21:39 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 7h<br>7h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | Darn-Write | neve | 6h<br>6h | batten, fabric |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | Darn-Write | neve | 1d<br>3h | fabric, cleanup, batten |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T21 | 1d<br>1d | docs |

## Sketches (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| S1 |  | **Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack** | [`self-healing-runs`](design-self-healing-runs.md)<br>[Sketch](decision-self-healing-runs.md) | T10<br>T12 | 3h<br>9m | dobby, batten, orch |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A catch-up fetch that fails on an unreachable origin still stops a run start**<br>• push-retry-not-halt made pushes retry quietly, but dobby catch-up fetch failure still refuses the start | [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md) | 9m<br>9m | dobby, fabric |
| B2 |  | **A missing local origin is not classed as unreachable by push or origin reconcile** | [`missing-origin-unreachable`](design-missing-origin-unreachable.md) | 0m<br>0m | fabric, git |

## Enhancements (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests still hand-write MailAgent closures and envkit session stand-ins**<br>• Move the MailAgent rows to a tier1 test over a real creel daemon and drop the envkit stand-ins | [`pr-session-mail-stand-ins`](design-pr-session-mail-stand-ins.md) | 2h<br>10m | testkit, landing |
| E2 |  | **Arras still shows in seeded profiles and in the overview and glossary**<br>• The landed design-first route left arras keys in the profile template and descriptions in two docs | [`arras-visible-leftovers`](design-arras-visible-leftovers.md) | 1h<br>1h | board, docs |
| E3 |  | **The recorded tmux kit cannot replay a pasted call, so no scenario that pastes runs over it**<br>• tmuxrec neither records nor matches stdin, and each paste names a fresh random buffer, so a replayed paste fails as unanswered | [`tmuxrec-paste-replay`](design-tmuxrec-paste-replay.md) | 1h<br>1h | testkit, reed |
| E4 | L | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | 14h<br>3h | landing, gate, batten |
| E5 |  | **internal/shedcli.names is reached by no main package**<br>• Remove the function or give it a caller; TestTestOnlyCode fails on it in the upkeep gate | [`shedcli-names-unreached`](design-shedcli-names-unreached.md) | 10m<br>10m | shed |
| E7 |  | **No gate checks Go formatting**<br>• A mis-formatted Go file passes every tier; add a gofmt check to tier0 | [`battencli-wire-test-gofmt`](design-battencli-wire-test-gofmt.md) | 1h<br>9m | gate |
