# P0 Root: entries under no project

Rendered 2026-11-27 20:27 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack** | [`self-healing-runs`](design-self-healing-runs.md) | motif | Discussion-Write | neve | 2h<br>6m | dobby, batten, orch |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 5h<br>5h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | T11 | 5h<br>5h | batten, fabric |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 1d<br>2h | fabric, cleanup, batten |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 1d<br>1d | docs |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **battencli wire_test.go holds a line gofmt would rewrite, and no gate caught it**<br>• err =c.env.SeedChild.WriteSeed in internal/battencli/wire_test.go | [`battencli-wire-test-gofmt`](design-battencli-wire-test-gofmt.md) | 34m<br>34m | gate, batten |

## Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests still hand-write MailAgent closures and envkit session stand-ins**<br>• Move the MailAgent rows to a tier1 test over a real creel daemon and drop the envkit stand-ins | [`pr-session-mail-stand-ins`](design-pr-session-mail-stand-ins.md) | T11 | 1h<br>1h | testkit, landing |
| E2 |  | **Arras still shows in seeded profiles and in the overview and glossary**<br>• The landed design-first route left arras keys in the profile template and descriptions in two docs | [`arras-visible-leftovers`](design-arras-visible-leftovers.md) |  | 34m<br>34m | board, docs |
| E3 |  | **The recorded tmux kit cannot replay a pasted call, so no scenario that pastes runs over it**<br>• tmuxrec neither records nor matches stdin, and each paste names a fresh random buffer, so a replayed paste fails as unanswered | [`tmuxrec-paste-replay`](design-tmuxrec-paste-replay.md) |  | 0m<br>0m | testkit, reed |
| E4 | L | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) |  | 13h<br>2h | landing, gate, batten |
