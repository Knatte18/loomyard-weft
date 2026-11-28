# P0 Root: entries under no project

Rendered 2026-11-28 08:15 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _on_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (6)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub** | [`kde-window-placement-build`](design-kde-window-placement-build.md)<br>[Sketch](decision-kde-window-placement-build.md) | tapestry · solid | Webster | neve | 17h<br>17h | termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs |
| T9 | **boardengine: TestDiscussedPayload crashes with concurrent map writes** | [`discussed-payload-map-race`](design-discussed-payload-map-race.md) | darn · light | landing · Finalize | neve | 10h<br>10h | board |
| T10 | **Worktree-Create blocks on a transient git ls-remote network error, and the stop question misreads a fresh branch's absence** | [`worktree-create-transient-net`](design-worktree-create-transient-net.md) | darn · light | Code-Burler-Fix | neve | 16h<br>16h | batten, fabric |
| T19 | **Timing a gate test call includes the slot wait; run time needs the run record** | [`gate-call-shows-slot-wait`](design-gate-call-shows-slot-wait.md) | darn · light | Code-Burler-Fix | neve | 10h<br>10h | gate |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | Darn-Write | neve | 1d<br>13h | fabric, cleanup, batten |
| T22 | **internal/shedcli.names is reached by no main package**<br>• Remove the function or give it a caller; TestTestOnlyCode fails on it in the upkeep gate | [`shedcli-names-unreached`](design-shedcli-names-unreached.md) | darn · light | landing · Finalize | neve | 10h<br>17m | shed |

## Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T20 | **webster reset refuses before its records push when fabricSync fails after the records commit landed** | [`reset-partial-commit-push`](design-reset-partial-commit-push.md) | darn · light |  | 18m<br>18m | webster, fabric |

## Layer A (7)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **A missing local origin is not classed as unreachable by push or origin reconcile** | [`missing-origin-unreachable`](design-missing-origin-unreachable.md) | darn · light | T21 | 10h<br>10h | fabric, git |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T21 | 1d<br>1d | docs |
| T4 | **Finalize: a parent ahead by owed landing tips loses every race and verifies under the parent lock** | [`finalize-owed-tips-gate`](design-finalize-owed-tips-gate.md) | darn · light | T12 | 10h<br>10h | landing |
| T8 | **runpusher tries to start pushers for removed task worktrees** | [`runpusher-removed-worktrees`](design-runpusher-removed-worktrees.md) | darn · light | T21 | 10h<br>10h | fabric |
| T11 | **Discussion-Bouncer reports stalemate when the fixer's decision-record edits are uncommitted** | [`bouncer-stalemate-uncommitted`](design-bouncer-stalemate-uncommitted.md) | darn · light | T12 | 10h<br>10h | shed, burler |
| T13 | **Reviews and plans check their claims against the code: fix keys, test infrastructure, card claims and first callers** | [`review-fix-claims-checked`](design-review-fix-claims-checked.md) | darn · light | T12 | 10h<br>10h | burler, stencil, webster |
| T18 | **Darn writer prompt forbids the gate calls a measurement entry requires** | [`darn-writer-measure-gates`](design-darn-writer-measure-gates.md) | darn · light | T12 | 10h<br>10h | darn, stencil |

## Sketches (1)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| S1 |  | **Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack** | [`self-healing-runs`](design-self-healing-runs.md)<br>[Sketch](decision-self-healing-runs.md) | T10<br>T12 | 14h<br>10h | dobby, batten, orch |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 10d<br>10d | shuttle |

## Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A catch-up fetch that fails on an unreachable origin still stops a run start**<br>• push-retry-not-halt made pushes retry quietly, but dobby catch-up fetch failure still refuses the start | [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md) | 10h<br>10h | dobby, fabric |

## Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **PR-Review session tests still hand-write MailAgent closures and envkit session stand-ins**<br>• Move the MailAgent rows to a tier1 test over a real creel daemon and drop the envkit stand-ins | [`pr-session-mail-stand-ins`](design-pr-session-mail-stand-ins.md) | 13h<br>10h | testkit, landing |
| E2 |  | **Arras still shows in seeded profiles and in the overview and glossary**<br>• The landed design-first route left arras keys in the profile template and descriptions in two docs | [`arras-visible-leftovers`](design-arras-visible-leftovers.md) | 12h<br>12h | board, docs |
| E3 |  | **The recorded tmux kit cannot replay a pasted call, so no scenario that pastes runs over it**<br>• tmuxrec neither records nor matches stdin, and each paste names a fresh random buffer, so a replayed paste fails as unanswered | [`tmuxrec-paste-replay`](design-tmuxrec-paste-replay.md) | 11h<br>11h | testkit, reed |
| E4 | L | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | 1d<br>13h | landing, gate, batten |
| E7 |  | **No gate checks Go formatting**<br>• A mis-formatted Go file passes every tier; add a gofmt check to tier0 | [`battencli-wire-test-gofmt`](design-battencli-wire-test-gofmt.md) | 12h<br>10h | gate |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator** | [`autonomy-intake-runs`](design-autonomy-intake-runs.md) | 10h<br>10h | board, orch, dobby |
