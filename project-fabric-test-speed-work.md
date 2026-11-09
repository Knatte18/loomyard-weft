# P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes

Rendered 2026-11-09 18:27 UTC; every age below is as of then.
Under: [Board](README.md).

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 2d · Owner `neve`

Groups the entries filed from the Sketch `fabric-test-speed`: hubforge-registered-templates, hubforge-slim-copy, git-config-from-go, dotgit-worktree-reader, hubreconcile-templates, loomcli-templates, battencli-templates, fabric-test-folds, fabric-anchor-origin-templates, fabric-livestate-merge-templates, copyhub-fixtures-and-hook, fabric-test-speed-remeasure.

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
| T2 | `draft` **Make the eight longest serial internal/fabricengine tests parallel**<br>• Move their package-level test state into parameters so the serial phase of tier 1 shrinks | [`fabric-serial-phase-tests`](design-fabric-serial-phase-tests.md) | darn · solid | T3 | 6h<br>5h | fabric, testkit |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 |  | **Fewer git processes in the serial phase of internal/fabricengine tier 1**<br>• About 65 thousand git processes, each slower as more run at once | [`fabric-serial-git-spawns`](design-fabric-serial-git-spawns.md) | 6h<br>6h | fabric, git |
