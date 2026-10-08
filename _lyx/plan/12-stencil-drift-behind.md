# Card 12 — A drift class for a worktree behind the build

**Create:**
- `plan:internal/stencilstore#Source` -> `type Source struct`
- `plan:internal/stencilstore#BuildAncestry` -> `type BuildAncestry int`
- `plan:internal/stencilstore#BuildAncestryUnknown` -> `const BuildAncestryUnknown BuildAncestry = iota`
- `plan:internal/stencilstore#BuildInHead` -> `const BuildInHead BuildAncestry = 1`
- `plan:internal/stencilstore#BuildNotInHead` -> `const BuildNotInHead BuildAncestry = 2`
- `plan:internal/stencilstore#driftBehind` -> `const driftBehind driftClass = 4`

**Edit:**
- `internal/stencilstore#Reconcile` -> `func Reconcile(baseDir string, registry Registry, mode Mode, source Source) ([]string, error)`
- `internal/stencilstore#ForceRefresh` -> `func ForceRefresh(baseDir string, registry Registry, source Source) ([]string, error)`
- `internal/stencilstore#warnPortBackDrift` -> `func warnPortBackDrift(baseDir string, registry Registry, source Source)`
- `internal/stencilstore#classifyPortBackDrift` -> `func classifyPortBackDrift(boardContent, sourceContent, embedded []byte, build func() BuildAncestry) (driftClass, string)`
- `internal/stencilstore#driftClass.String`
- `cmd/lyx#seedSubtree` -> `func seedSubtree(hub, baseDir, subtreeRel string, registry stencilstore.Registry, mode stencilstore.Mode, source stencilstore.Source, label string)`
- `cmd/lyx#seedStencilsAt`
- `internal/cliwire#Module.ResolveStandalone`
- `internal/stencilcli#Command`
- `internal/testkit/stencilkit#SeedInto`
- `internal/stencilstore/drift_test.go`
- `internal/stencilstore/reconcile_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/stencilcli/cli_integration_test.go`

**Intent:**
Decision 12 inside `internal/stencilstore`, which runs no git (Decision 13): the port-back drift warning learns whether the worktree holds the binary's build commit, as an input the caller supplies, and stops telling a worktree that lacks it to deploy (issue #445).

`Source` replaces the bare `sourceDir string` parameter.
- `Dir` is the worktree source tree; empty names none and keeps the warning silent, as `""` did.
- `Build` is a `func() BuildAncestry` that reports whether the worktree's HEAD holds the running binary's build commit; nil means unknown.

`BuildAncestry` has three values:
- `BuildAncestryUnknown`: the binary carries no VCS revision, or the read failed.
- `BuildInHead`.
- `BuildNotInHead`.

`classifyPortBackDrift` keeps its hand-edited, both and neither cases unchanged.
In today's source-ahead case (board copy untouched, source differs from the embedded bytes) it calls `build`, once, and only there, so a stencil with no drift, or any other class, never reaches it:
- `BuildNotInHead` classifies `driftBehind`, with the message: the board copy differs from the worktree source because this worktree is behind the build that deployed it; syncing this worktree with main resolves it. No deploy advice, no `update-plugins.sh`.
- `BuildInHead`, `BuildAncestryUnknown` and a nil `build` keep `driftSourceAhead` and its deploy advice, as today.

`driftClass.String` names the new class `behind`.
`warnPortBackDrift` reads `source.Dir` and passes `source.Build` through.
It logs a `behind` class with `logger.Info`, since it is advice that clears by itself on a sync, and every other class with `logger.Warn`, as today.
`Reconcile` and `ForceRefresh` take `Source` and call the warning only when `source.Dir` is non-empty.
Their doc comments and `Reconcile`'s file header mention the input.

Callers pass what they passed before, wrapped:
- `seedSubtree` takes a `stencilstore.Source`.
- `seedStencilsAt` builds `stencilstore.Source{Dir: sourceDir}` for the stencils pass and `stencilstore.Source{}` for the specs pass.
- `stencilcli`'s sync verb passes `stencilstore.Source{Dir: sourceDir}` and `stencilstore.Source{}`.
- `cliwire`'s two calls, `stencilkit.SeedInto` and the three tests' direct calls pass `stencilstore.Source{}`.
No behavior changes outside `stencilstore` in this card; card 13 supplies the ancestry.

Tests:
- `drift_test.go`'s `TestClassifyPortBackDrift` table gains a `build` column, counting its calls.
  - Source-ahead with `BuildNotInHead` gives `behind`, with "syncing" in the message and neither `update-plugins.sh` nor `promote`.
  - With `BuildInHead`, and with a nil `build`, it stays `source-ahead` with `update-plugins.sh`.
  - The hand-edited, both and neither rows assert zero calls.
- `TestWarnPortBackDrift_EmitsClassAndRemedyPerDifferingStencil` gains a stencil that drifts as behind and asserts its line carries `level=INFO` and class `behind`.
  It also asserts the build func was never called for the equal stencil (a worktree with no drift runs no ancestry read).
- `reconcile_test.go` and the three other test files only switch to the `Source` argument.

**ImpactSummary:** Re-signs stencilstore's two seed entry points and its two drift helpers; their six production callers and five test files pass a wrapped `Source` with today's directory, so only stencilstore's own classification changes.

**Commit:** 12: stencilstore: classify a worktree behind the build as behind, with no deploy advice
