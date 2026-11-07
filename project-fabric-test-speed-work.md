# P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes

Rendered 2026-11-07 15:11 UTC; every age below is as of then.
Under: [Board](README.md).

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 55m · Owner `neve`

Groups the entries filed from the Sketch `fabric-test-speed`: hubforge-registered-templates, hubforge-slim-copy, git-config-from-go, dotgit-worktree-reader, hubreconcile-templates, loomcli-templates, battencli-templates, fabric-test-folds, fabric-anchor-origin-templates, fabric-livestate-merge-templates, copyhub-fixtures-and-hook, fabric-test-speed-remeasure.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (10)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **A slimmer hubforge.CopyHub: packed templates, no walk per copy, two-process binding fold** | [`hubforge-slim-copy`](design-hubforge-slim-copy.md)<br>[Sketch](decision-hubforge-slim-copy.md) | darn · light | Code-Burler-Review | neve | 55m<br>55m | fabric, testkit, git |
| T10 | **internal/hubreconcile fixtures onto registered templates and CopyHub, with the held-lock test at tier 0** | [`hubreconcile-templates`](design-hubreconcile-templates.md)<br>[Sketch](decision-hubreconcile-templates.md) | darn · light | Darn-Write | neve | 55m<br>55m | fabric, testkit, git |
| T11 | **internal/loomcli beamHub and newBadReedUpFixture onto registered templates** | [`loomcli-templates`](design-loomcli-templates.md)<br>[Sketch](decision-loomcli-templates.md) | darn · light | Describe | neve | 55m<br>55m | fabric, testkit, git |
| T12 | **internal/battencli beamHub onto a registered template** | [`battencli-templates`](design-battencli-templates.md)<br>[Sketch](decision-battencli-templates.md) | darn · light | Code-Burler-Fix | neve | 55m<br>55m | fabric, testkit, git |
| T13 | **Fold the overlapping internal/fabricengine tests the tier 1 report read** | [`fabric-test-folds`](design-fabric-test-folds.md)<br>[Sketch](decision-fabric-test-folds.md) | darn · light | Darn-Write | neve | 55m<br>55m | fabric, testkit, git |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **gitkit and hubforge initialisers write their per-repository git config from Go** | [`git-config-from-go`](design-git-config-from-go.md)<br>[Sketch](decision-git-config-from-go.md) | darn · light | T6 | 55m<br>55m | fabric, testkit, git |
| T14 | **internal/fabricengine anchor and origin fixtures onto registered templates** | [`fabric-anchor-origin-templates`](design-fabric-anchor-origin-templates.md)<br>[Sketch](decision-fabric-anchor-origin-templates.md) | darn · light | T13 | 55m<br>55m | fabric, testkit, git |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **internal/fabricengine live-state arranges and merge-pair layers onto registered templates** | [`fabric-livestate-merge-templates`](design-fabric-livestate-merge-templates.md)<br>[Sketch](decision-fabric-livestate-merge-templates.md) | darn · light | T13<br>T14 | 55m<br>55m | fabric, testkit, git |

### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T16 | **CopyHub in the NewHub fixtures of fabricengine and adoptcli, and the hook without dirname and basename** | [`copyhub-fixtures-and-hook`](design-copyhub-fixtures-and-hook.md)<br>[Sketch](decision-copyhub-fixtures-and-hook.md) | darn · light | T15 | 55m<br>55m | fabric, testkit, git |

### Layer D (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Re-measure tier 1 and internal/fabricengine after the fabric test speed tasks land** | [`fabric-test-speed-remeasure`](design-fabric-test-speed-remeasure.md)<br>[Sketch](decision-fabric-test-speed-remeasure.md) | darn · docs | T16<br>T7<br>T10<br>T11<br>T12 | 55m<br>55m | fabric, testkit, git |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
