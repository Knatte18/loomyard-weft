# P2 Codex CLI as a second shuttle engine: codexengine

Rendered 2026-11-03 17:08 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine`](design-codex-engine.md) · Age 4w · Owner `neve`

codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs

beam

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T13 | **Merge main into the codex-engine beam again and resolve every conflict**<br>• Bring main into the beam before it lands, so its landing merges cleanly | [`codex-engine-merge-main-2`](design-codex-engine-merge-main-2.md) | darn | Drift-Verify | neve | 12m<br>12m | codex, shuttle |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
