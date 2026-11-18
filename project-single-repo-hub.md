# P4 Single-repository hubs: records on each code branch, the board on lym-bolt

Rendered 2026-11-18 19:58 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 5m · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay** | [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md)<br>[Sketch](decision-single-repo-fabric-mode.md) | tapestry · solid |  | 5m<br>5m | fabric, cwd, cli, landing, docs, pattern |
| T6 | `draft` **Records-free code identity and pull request views for one-repository hubs** | [`single-repo-code-identity`](design-single-repo-code-identity.md)<br>[Sketch](decision-single-repo-code-identity.md) | tapestry · solid | T5<br>T2 | 5m<br>5m | fabric, cwd, gate, git, weaver, drift, webster, lint, loom, landing, pawl, shed, arbiter |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
