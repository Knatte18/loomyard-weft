# Card 11 — package-load-tags

**Edit:**
- `internal/impactset#listPackages`
- `internal/impactset#ModuleGuardTests`
- `internal/planglyph#typesLoader.load`
- `internal/impactset#TestDerive_OverARepository`
- `internal/impactset/doc.go`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Every Go package load that lists test files lists them under the tier tags and the old spellings alike (D5, D13): `tier0,tier1,tier2,tier3,integration,tmux,llm`.

`impactset`'s reverse-dependency graph (`listPackages`) and guard-package listing (`ModuleGuardTests`) and `planglyph`'s type load (`typesLoader.load`) move from `integration,tmux,llm` to that set, so a scope sees a `tier0` test file's imports, and a branch not yet holding the retag still has its old-tagged files seen until D13's follow-up drops the old spellings.
`internal/planparser` runs no package load, so its module-wide check needs no change here; card 12 extends its tag guard.

`TestDerive_OverARepository` gains a row whose fixture package is reached only through a test file tagged `//go:build tier0`, asserting a diff of the imported package puts the importing package in the set.
The impactset and planglyph docs and `loom-plan-spec.md`'s `caller-uncovered` row name the new tag set.

**ImpactSummary:** Widens three `go list` tag sets; the impacted set, the guard listing and the plan gate's type load gain the tier-tagged files.
