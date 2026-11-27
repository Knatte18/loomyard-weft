# P0 Root: entries under no project

Rendered 2026-11-27 09:05 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Autonomy _off_ (hub-wide) · PR setting _off_ (hub-wide)

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Memory guard fails a gate by killing cmd/lym tier1 when the user slice is full from elsewhere** | [`memory-guard-infra-stop`](design-memory-guard-infra-stop.md) | darn · light | landing · Finalize | neve | 1h<br>1h | gate |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Command-help byte-cap test is not run by card or batch gates** | [`cli-form-test-in-card-gate`](design-cli-form-test-in-card-gate.md) | darn · light | T1 | 1h<br>1h | gate, cli |

## Layer H (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T21 | **Faults the hardener study read in fabric, cleanup and teardown code** | [`hardener-study-faults`](design-hardener-study-faults.md) | darn · light | T11 | 21h<br>19h | fabric, cleanup, batten |

## Layer I (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys** | [`config-key-comment-sweep`](design-config-key-comment-sweep.md) | darn · light | T11<br>T21 | 19h<br>12h | docs |

## Layer J (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Finalize blames the fixer session for a push of origin/<task branch> it did not make** | [`finalize-fixer-halt-blame`](design-finalize-fixer-halt-blame.md) | darn · light | T9<br>T17 | 1h<br>1h | landing |

## Layer K (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Finalize records a DNS failure classed transient as a failed / producer-hard-failure halt** | [`transient-stop-not-failed`](design-transient-stop-not-failed.md) | darn · light | T2 | 1h<br>1h | shed, loom |

## Draft (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | `draft` **Remove the coverage maps and every user of them, keeping the redundancy sweep and red-first** | [`remove-testmap`](design-remove-testmap.md) | darn · solid | T1<br>T8 | 1h<br>1h | testmap, gate, darn, loom, stencil, cleanup |

## Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 9d<br>9d | shuttle |

## Bugs (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| B1 |  | **A rejected push can force origin past a commit made by hand on GitHub** | [`push-over-github-hand-commit`](design-push-over-github-hand-commit.md) | T23 | 1h<br>1h | fabric, landing |
| B2 |  | **A computed or base gate scope skips the guard tests of an impacted package whose tier it does not run** | [`scope-skips-impacted-guards`](design-scope-skips-impacted-guards.md) |  | 1h<br>1h | gate, testkit |

## Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E1 |  | **Slow whole-repo guards and .NET builds leave the task gates for landing or periodic upkeep** | [`whole-repo-guards-off-gates`](design-whole-repo-guards-off-gates.md) |  | 11h<br>1h | gate, testkit |
| E2 |  | **Cross-machine mail into the receiving orch own channel instead of a channel per orch pair** | [`bridge-inbound-per-orch`](design-bridge-inbound-per-orch.md) |  | 19h<br>19h | slack, creel |
| E3 | H | **Finalize gates the merged tree without the parent lock and holds the lock only to land** | [`landing-lock-only-for-land`](design-landing-lock-only-for-land.md) |  | 2h<br>2h | landing, gate |
| E4 |  | **A landing queue gates several ready runs as one batch and lands them together** | [`landing-queue`](design-landing-queue.md) | E3 | 2h<br>2h | landing, gate, batten |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs** | [`kde-window-placement`](design-kde-window-placement.md) | 11h<br>11h | termwindow, desktop |
