# P4 Single-repository hubs: records on each code branch, the board on lym-bolt

Rendered 2026-11-19 11:30 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 15h · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Records-free code identity and pull request views for one-repository hubs** | [`single-repo-code-identity`](design-single-repo-code-identity.md)<br>[Sketch](decision-single-repo-code-identity.md) | tapestry · solid | T7 | 15h<br>2h | fabric, cwd, gate, git, weaver, drift, webster, lint, loom, landing, pawl, shed, arbiter |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T5 | **One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay** | [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md)<br>[Sketch](decision-single-repo-fabric-mode.md) | 15h<br>15h | fabric, cwd, cli, landing, docs, pattern |
