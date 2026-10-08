# Plan gate: typed caller scan, nested-module gates, admissible fixes

The plan gate no longer reports callers that are not callers, and no longer lets a plan through whose gate cannot pass.
The plan reviewer stops proposing fixes the gate blocks.

Closes #461
Closes #451
Closes #460

## What changes

- `caller-uncovered` (`internal/planglyph`) resolves references by type instead of by name.
  A re-signed method matches only calls and method values on its own receiver type, promoted calls through embedding included.
  A call on another type with a same-named method, a package-qualified `pkg.M`, and a call through an interface are no longer reported.
  A package-level member is matched by import path, so aliased and dot imports are now caught.
- A gate command that tests packages inside a nested Go module runs inside it.
  Webster's Go-derived card gate emits `go -C <module> test -tags integration ./<module-relative-dir>` per nested module.
  The new blocking plan check `verify-nested-module` (`internal/planparser`) refuses an author-written `**Verify:**` value or `## verify:` line that tests a nested module's package from outside it.
  It runs wherever planparser's format checks run, `lyx loom validate-plan` included.
- The Plan-Review rubric gets a `## Proposed fixes` rule: a proposed fix must pass every check in the plan spec's validation-checks section, and a nested module's packages count as covered by a command run with `go -C <module>`.
  The plan template states the `go -C` form.
- `CLAUDE.md` gains a `## Test budget` section: a card whose new tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- `contracts/specs/loom-plan-spec.md`, `internal/planglyph/doc.go` and `internal/websterengine/doc.go` describe the new behavior; the old bound about aliased and dot imports is gone from the `caller-uncovered` row.

## Design decisions

- Types come from `golang.org/x/tools/go/packages`, a new direct dependency pinned at `v0.42.0`; the offline module cache held no newer usable version, and `golang.org/x/mod` ended at `v0.34.0` with it.
  The token pre-scan still picks candidate files; only their directories and the subjects' declaring directories are loaded, with test variants and the tags `integration,tmux,llm`.
- The load runs offline (`GOPROXY=off`, `-mod=readonly`), under a 3-minute timeout, logged through `internal/logger`, and behind a `typesLoader` seam so untagged tests inject it.
  Where the load fails or leaves a file or identifier untyped, the scan falls back to import-path matching.
  A load failure is never `ErrQuarryUnavailable`.
- Severity follows resolution.
  Type-resolved and import-resolved references are blocking, method callers included, which is new; plans that passed before can now get a blocking method finding.
  Name-only method matches collapse into one informational finding per subject, naming files and lines and stating that the receiver was unresolved.
- A nested module's files are scanned through the fallback, because the load runs from the root module.
  A package-level caller there is blocking; a method caller there is informational.
- `planparser.NestedModules` and `ModuleOf` are the single derivation of a plan's nested modules, shared by the card gate and `verify-nested-module`.
  A module counts if its `go.mod` exists on disk or a `Create` target, or a Rename pair, of a card at or before the one in question brings it in.
  A `Move` relocating a `go.mod` is not counted, so for a later card the verdict can differ between the plan gate and dispatch; moving a `go.mod` through a Rename pair avoids it.
- `verify-nested-module` only refuses.
  It handles a preceding `cd <dir>`, `go -C <dir>` and `-C=<dir>`, `&&`/`;` separators, env assignments and `-args`, and it ignores flag values such as `-o ./bin`.
  It misses commands built from variables, inside `bash -c`, after a subshell, or with an import-path package argument; webster's gates still catch those at run time.
- A method subject's coverage record now also carries its declaring package's directory, clause and import path, which the fallback needs.
- In `internal/loomcli` the parity fixture helper `fabricReferencePlanFixture` is renamed `verifySectionPlanFixture` and takes the verify body, so the new parity row reuses it.
- `internal/planglyph` raises its row in `cmd/lyx/testdata/test-budget.yaml` for `TestGlyphChain_CallerCoverageTyped`.

## To check by hand

- Plan-Review left one finding addressed but not re-reviewed: a nested-module test row in the `verify-nested-module` card contradicts the rule the check enforces (`card3-nested-module-test-row-contradicts-rule`).
  Read the new `verify-nested-module` rows in `internal/planparser/validate_test.go` against `contracts/specs/loom-plan-spec.md`.
- A cold build cache makes the first type load compile the dependency graph, tree-sitter's cgo grammars included.
  A timeout then degrades method findings to informational for that gate run; time a plan gate on a cold cache.
- A badly broken package yields mostly name-only matches; confirm the fallback finding reads sensibly on one.
- The integration tier (`go test -tags integration ./...`) is the only place the real `go list` load runs; run it before merging.
