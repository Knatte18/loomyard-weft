# Plan gate checks: nested modules and import-path caller scan

Both batches recorded `done`, with no `stuck`, `failed` or recovery runs.

## Batch 01-nested-modules (head `4aae462da`)

- Card 1: `planparser` gained `NestedModules` and `ModuleOf`, the single derivation of a plan's nested modules.
- Card 2: webster's card gate tests a nested module's packages with `go -C <module>`; `TestCardGateCommand` and the docs and spec follow.
- Card 3: the blocking plan check `verify-nested-module` is wired into `validate` after `commit-subject-mismatch`, with its spec row inserted and later rows renumbered.
- Deviation reported by the fork: the `internal/loomcli` parity fixture helper `fabricReferencePlanFixture` was renamed `verifySectionPlanFixture` and given a verify-body parameter so the new parity row reuses it.
  Several helper symbols in `planparser` and `websterengine` were also flagged as informational scope-outside-plan warnings.

## Batch 04-caller-scan-import-path (head `4215dc083`)

- Card 4: `caller-uncovered` resolves callers by import path (aliased and dot imports included), collapses name-only method matches into one informational finding, and reaches nested modules.
- Card 5: a `go/packages` type load sits in front of the scan through a `typesLoader` seam, with the import-path scan as the fallback; `TestGlyphChain_CallerCoverageTyped` raises the `internal/planglyph` budget row.
- Card 6: Plan-Review rubric gained `## Proposed fixes`, and the plan template states the `go -C` form.
- Card 7: `CLAUDE.md` gained a `## Test budget` section.
- Deviations reported by the fork: `golang.org/x/tools` was pinned at `v0.42.0` and `golang.org/x/mod` ended at `v0.34.0`, because the offline module cache had no usable newer version; a method subject's coverage now also carries its declaring package's directory, clause and import path.
  Many helper symbols in `planglyph` were flagged as informational scope-outside-plan warnings.
