# P2 Stress test of codexengine: every designed capability, hooks above all

Rendered 2026-11-03 19:27 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 16m · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Codex trial: home, trust, sandbox and refusals**<br>• Seven cases on real Codex launches against the package doc | [`codex-trial-home-sandbox`](design-codex-trial-home-sandbox.md) | darn · docs |  | 0m<br>0m | codex, shuttle |
| T5 | `draft` **Codex trial: hooks, shell guards and skills**<br>• Seven cases on real Codex launches against the package doc | [`codex-trial-hooks-guards-skills`](design-codex-trial-hooks-guards-skills.md) | darn · docs | T3 | 0m<br>0m | codex, shuttle |
| T6 | `draft` **Codex trial: pane, sends, creel by the pane path, and load**<br>• Six cases, three sessions at once and one darn run with a Codex reviewer seat | [`codex-trial-pane-load`](design-codex-trial-pane-load.md) | darn · docs | T5 | 0m<br>0m | codex, shuttle, creel |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
