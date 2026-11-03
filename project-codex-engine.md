# P2 Codex CLI as a second shuttle engine: codexengine

Rendered 2026-11-03 13:29 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine`](design-codex-engine.md) · Age 4w · Owner `neve`

codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs

beam

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **codexengine: the Codex CLI shuttle engine, its home and its launch** | [`codex-engine-core`](design-codex-engine-core.md) | darn · solid | Code-Bouncer | neve | 50m<br>50m | shuttle, codex |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Skills and every shell guard row on Codex** | [`codex-skills-guards`](design-codex-skills-guards.md) | darn · light | T1 | 50m<br>50m | shuttle, codex |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **The assistant seat and the Burler review round with a Codex assistant** | [`codex-assist-seat`](design-codex-assist-seat.md) | darn · solid | T1<br>T3 | 50m<br>50m | burler, shuttle, codex |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
