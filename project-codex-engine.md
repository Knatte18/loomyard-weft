# P2 Codex CLI as a second shuttle engine: codexengine

Rendered 2026-11-03 12:48 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine`](design-codex-engine.md) · Age 4w · Owner `neve`

codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs

beam

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (4)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **codexengine: the Codex CLI shuttle engine, its home and its launch** | [`codex-engine-core`](design-codex-engine-core.md) | darn · solid |  | 10m<br>10m | shuttle, codex |
| T3 | `draft` **Skills and every shell guard row on Codex** | [`codex-skills-guards`](design-codex-skills-guards.md) | darn · light | T1 | 10m<br>10m | shuttle, codex |
| T4 | `draft` **Engine-keyed stencil blocks and heddle's codex-file fallback** | [`codex-stencil-blocks`](design-codex-stencil-blocks.md) | darn · light |  | 10m<br>10m | stencil, heddle, codex |
| T5 | `draft` **The assistant seat and the Burler review round with a Codex assistant** | [`codex-assist-seat`](design-codex-assist-seat.md) | darn · solid | T1<br>T3<br>T4 | 10m<br>10m | burler, shuttle, codex |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
