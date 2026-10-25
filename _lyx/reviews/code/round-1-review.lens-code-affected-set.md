```yaml
lens: code-affected-set
outcome: reported
```

Task: list, per changed surface, the set of affected callers, tests, docs and stencils, and flag the members the diff left wrong.

## Per-change affected set

| Change | Set found | Covered? |
|---|---|---|
| `--window` / `--open-terminal` retired, `ResumeInWindow` removed | Grep for `--window`, `open-terminal`, `OpenTerminal` and `ResumeInWindow` over `*.go`, `*.md` and `*.yaml` found nothing outside `_lyx` and `.lyx`. The orch stencil dropped the start steps. | Covered |
| Ready snapshot, `ReadyRoutes` and the `newly ready` line dropped | The only remaining hit is a creel delivery fixture string (`internal/creelengine/deliver_test.go:424,433`). It is opaque message text, so it is not stale. | Covered |
| `Yield` outcome: `shedengine.Yield`, `RunYielded`, `ExitYield`, step envelope `continue` | Producers that return Stuck or Done are unaffected. The envelope `continue` change in `shedverbs/step.go` and the `sheddriver/drive.go` yield branch are both in the diff. | Covered |
| Arbiter keys moved to `shed.yaml` with module overrides | Resolver readers: `arbiterstop/holdreading.go` was moved to `sheddriver.ArbiterLimits` and `loomengine.RecipeOverride`. The `darnengine`, `battenshed` and `loomengine` templates and configs gained the override keys. `configreg/keymove.go` carries the loom to darn copy. `loomcli/doc.go` and `sheddriver/doc.go` are updated. | Covered, bar F1 |
| Draft status: writers of a task's status | `pairteardown/claim.go:108`, `battencli/cancelcmd.go:461`, `batten startguard.go:95` and `render.go:508` all handle `draft`. The remaining `Status = nil` at `boardengine/ownerset.go:223` is `turnIntoProject`, which makes the entry a project, so the "no lyx path writes a null task status" rule holds. | Covered |
| Orch role stencil | The stencil drops the start steps. Line 141 says dobby starts ready tasks. | Covered |
| New `.lyx/beam/` directory | `lyxdirs.BeamDirName` is added and `dobbyengine` is added to the cwd-resolution bound-packages list. | Covered |

## Findings

**F1 — LOW, class consistency**
- Location: `internal/battenshed/deps.go:117-121`.
- Summary: the `ChildArbiterStop` field docs still say `On` and `Timeout` are "read from the child's loom config", but `HoldReading` now resolves them from `shed.yaml` plus the recipe's override.
- Evidence: `internal/arbiterstop/holdreading.go:65-86` calls `sheddriver.LoadConfig`, `loomengine.RecipeOverride` and `sheddriver.ArbiterLimits`. The `deps.go` comments were not touched by the diff.
- Fix: reword the two field comments to name the shed arbiter keys with the recipe's override.

I found no other member of any affected set that the diff leaves wrong. I did not check stencils other than `orch-template-role.md`, nor whether `docs/overview.md` still describes batten's wait or `--window`.

## Tool calls
1. Grep for `--window`, `open-terminal`, `ReadyRoutes`, `umbrella` and the arbiter keys across the tree and docs. My context only held the diff, not the other references.
2. Grep for Stuck/Done outcome consumers, nil status writers and empty-status readers. My context lacked the other callers.
3. Read `boardengine/ownerset.go` 205-235, `battenshed/deps.go` 108-130 and the `cancelcmd.go` diff, to check each nil status write and the arbiter `On` and `Timeout` doc comments.
4. Grep for `ChildArbiterStop` construction sites and the orch stencil's `landed`/`ready` wording, to check the stencil and the arbiter reading path.
5. Read the `holdreading.go` diff and lines 60-100, to confirm the arbiter reading moved to shed config.
