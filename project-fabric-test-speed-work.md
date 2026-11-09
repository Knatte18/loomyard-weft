# P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes

Rendered 2026-11-09 19:14 UTC; every age below is as of then.
Under: [Board](README.md).

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 2d · Owner `neve`

Groups the entries filed from the Sketch `fabric-test-speed`: hubforge-registered-templates, hubforge-slim-copy, git-config-from-go, dotgit-worktree-reader, hubreconcile-templates, loomcli-templates, battencli-templates, fabric-test-folds, fabric-anchor-origin-templates, fabric-livestate-merge-templates, copyhub-fixtures-and-hook, fabric-test-speed-remeasure.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Make the eight longest serial internal/fabricengine tests parallel**<br>• Move their package-level test state into parameters so the serial phase of tier 1 shrinks | [`fabric-serial-phase-tests`](design-fabric-serial-phase-tests.md) | darn · solid | Code-Burler-Review | neve | 7h<br>6h | fabric, testkit |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Fewer git processes in the serial phase of internal/fabricengine tier 1**<br>• About 65 thousand git processes, each slower as more run at once | [`fabric-serial-git-spawns`](design-fabric-serial-git-spawns.md) | darn · solid | T2 | 7h<br>33m | fabric, git, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
