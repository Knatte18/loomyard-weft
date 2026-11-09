# P4 Arras: one recipe that runs Motif and raddle in a row, with a perch on the split

Rendered 2026-11-09 14:29 UTC; every age below is as of then.
Under: [Board](README.md).

[`arras-recipe`](design-arras-recipe.md) · Age 22h · Owner `osl-1046`

Groups the entries filed from the Sketch `motif-raddle-recipe`: project-release-members, raddle-perch-mode, arras-recipe-perch, arras-family-filing, arras-shipped-text.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **The arras recipe and its split perch**<br>• `loom: code_assist_reviewer_model astra[medium]` | [`arras-recipe-perch`](design-arras-recipe-perch.md)<br>[Sketch](decision-arras-recipe-perch.md) | darn · solid | Darn-Write | osl-1046 | 22h<br>22h | shed, loom, burler, arbiter, board |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **An arras family filing turns the run's entry into its project and re-points its dependents** | [`arras-family-filing`](design-arras-family-filing.md)<br>[Sketch](decision-arras-family-filing.md) | darn · solid | T6 | 22h<br>22h | shed, loom, burler, arbiter, board |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **The shipped rule and docs name arras as the recipe for a tapestry-sized note** | [`arras-shipped-text`](design-arras-shipped-text.md)<br>[Sketch](decision-arras-shipped-text.md) | darn · docs | T6<br>T7 | 22h<br>22h | shed, loom, burler, arbiter, board |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
