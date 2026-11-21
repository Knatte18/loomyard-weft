# P3 Put the per-test coverage maps to use

Rendered 2026-11-21 13:35 UTC; every age below is as of then.
Under: [Board](README.md).

[`testmap-uses`](design-testmap-uses.md) · Age 2d · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

The uses the map study ranked worth building (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, `884424fb`).
What only the maps give is the lookup between a function and the tests that reach it, across packages; none of the uses saves test time.

Build order: the reach note first, since the other two read the maps through its reader; then failure triage; then the diff mutation check, starting with its hand trial.
The maps need no upkeep per landing for these: a stored map stays sound for days, and the packages a diff touches are mapped fresh in seconds.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **Measure whether the reach and mutation reports change what runs land**<br>• Over 10 landed darn runs, count how often the reach note, reach turn and mutation survivors led to added tests or fixes<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`testmap-use-measure`](design-testmap-use-measure.md) | darn · docs |  | 2d<br>2d | testmap, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
