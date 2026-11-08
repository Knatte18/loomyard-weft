# P4 Arras: one recipe that runs Motif and raddle in a row, with a perch on the split

Rendered 2026-11-08 19:08 UTC; every age below is as of then.
Under: [Board](README.md).

[`arras-recipe`](design-arras-recipe.md) · Age 2h · Owner `neve`

Groups the entries filed from the Sketch `motif-raddle-recipe`: project-release-members, raddle-perch-mode, arras-recipe-perch, arras-family-filing, arras-shipped-text.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Releasing a project with lym board set-status releases its draft member tasks** | [`project-release-members`](design-project-release-members.md)<br>[Sketch](decision-project-release-members.md) | darn · light |  | 2h<br>2h | shed, loom, burler, arbiter, board |
| T5 | `draft` **Raddle's perch mode: recorded from the recipe's rows, with its outcome checks and refusals** | [`raddle-perch-mode`](design-raddle-perch-mode.md)<br>[Sketch](decision-raddle-perch-mode.md) | darn · solid |  | 2h<br>2h | shed, loom, burler, arbiter, board |
| T6 | `draft` **The arras recipe and its split perch** | [`arras-recipe-perch`](design-arras-recipe-perch.md)<br>[Sketch](decision-arras-recipe-perch.md) | darn · solid | T5 | 2h<br>2h | shed, loom, burler, arbiter, board |
| T7 | `draft` **An arras family filing turns the run's entry into its project and re-points its dependents** | [`arras-family-filing`](design-arras-family-filing.md)<br>[Sketch](decision-arras-family-filing.md) | darn · solid | T4<br>T5<br>T6 | 2h<br>2h | shed, loom, burler, arbiter, board |
| T8 | `draft` **The shipped rule and docs name arras as the recipe for a tapestry-sized note** | [`arras-shipped-text`](design-arras-shipped-text.md)<br>[Sketch](decision-arras-shipped-text.md) | darn · docs | T4<br>T5<br>T6<br>T7 | 2h<br>2h | shed, loom, burler, arbiter, board |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
