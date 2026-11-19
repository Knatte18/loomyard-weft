# P4 Single-repository hubs: records on each code branch, the board on lym-bolt

Rendered 2026-11-19 18:30 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 22h · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T7 | **One-repository hubs redesigned: records on their own branches, the layout known only to fabric**<br>• Redesign single-repo mode so records live on their own branches in the code repository, wired as _lym like weft, and decide what of f06acf39 stays | [`single-repo-records-branches`](design-single-repo-records-branches.md)<br>[Sketch](decision-single-repo-records-branches.md) | raddle | Raddle-File | neve | 6h<br>30m | fabric |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Remove f06acf39's records-on-code-branch layout and revert its two-repository widenings** | [`single-repo-code-records-removal`](design-single-repo-code-records-removal.md)<br>[Sketch](decision-single-repo-code-records-removal.md) | darn · solid |  | 0m<br>0m | fabric, cwd, cli, batten, loom, landing, git, board, docs, pattern |
| T9 | `draft` **One-repository hubs with records and board under origin's refs/lym/, the layout read only by fabric** | [`single-repo-namespace-layout`](design-single-repo-namespace-layout.md)<br>[Sketch](decision-single-repo-namespace-layout.md) | tapestry · solid | T8 | 0m<br>0m | fabric, git, docs, pattern |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
