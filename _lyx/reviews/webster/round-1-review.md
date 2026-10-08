---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/planglyph/typesload.go:129
    summary: a file with a parse error is marked checked from its partial AST, so a reference in the unparsed region is dropped instead of falling back to the scan
  - id: F2
    severity: LOW
    class: scope
    location: internal/planparser/verifymodule.go:79
    summary: verify-nested-module branches with no test row; the -C= spelling, the semicolon and newline separators, the leading env assignment, the -args cutoff, and a cd or -C to an absolute or out-of-root path
  - id: F3
    severity: LOW
    class: scope
    location: internal/planglyph/callercoverage.go:407
    summary: importsOf import-name rules have no test row; the major-version suffix drop, a subject package whose clause differs from its directory name, and a blank import
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/plan-gate-checks/status.json` gives `product.parent` as `main`, and no `_lyx/loom/rework/` directory exists.
I built the module, ran `go vet` under the untagged, `integration`, `tmux` and `llm` tags, `go test ./...`, and `go test -tags integration` over `planglyph`, `planparser`, `websterengine` and `loomcli`; all are green.
`go mod tidy -diff` is clean.

### [LOW] A file with a parse error is `checked` from its partial AST

**Location:** `internal/planglyph/typesload.go:129` (`typedReferences`), consumed by `combineReferences` in `internal/planglyph/callercoverage.go`.

**Issue:** `typedReferences` marks every file in `pkg.Syntax` as `checked` when its package carries `TypesInfo`.
`golang.org/x/tools/go/packages` returns "the ASTs of the ones that could be at least partially parsed" (`parseFiles` in `packages.go`), type-checks them, and records the parse error in `pkg.Errors`.
An identifier inside a region the parser dropped into a bad node is in neither `Uses` nor `Defs`, so it is not in `unresolved` either.
`combineReferences` then drops the scan's match on that line, because a checked file keeps only the lines `unresolved` records.
A reference there is lost, where card 5 states that the check "falls back to the import-path scan wherever type information is missing" and is "never weaker than card 4's" without types.
The gate runs on a tree that normally builds, so this is a narrow case; it is the one failure path I found where a reference is dropped rather than scanned.
I read the loader's source for this and did not run a broken-syntax fixture.

**Fix:** Leave a file out of `checked` when its package reports a `packages.ParseError` that names it, or when `pkg.Errors` holds any `ParseError`, so the scan decides the whole file.
Add a row to `TestGlyphChain_CallerCoverageTyped` with a syntax-broken file that holds a reference after the break, beside the existing `undefined()` row.

### [LOW] `verify-nested-module` branches with no test row

**Location:** `internal/planparser/verifymodule.go:79` (`crossModuleCommands`), `:122` (`goCommandDirAndPackages`), `:167` (`resolveRunDir`), `:180` (`isEnvAssignment`); tests in `internal/planparser/validate_test.go` (`TestValidate_VerifyNestedModule`).

**Issue:** `PATTERN-test-economy` and the rubric require every behavior the diff adds to have coverage.
`TestValidate_VerifyNestedModule` covers the rows card 3 lists.
These behaviors have no row, and a grep of the planparser tests finds no use of them:
- the `-C=<dir>` spelling;
- the `;` and newline separators;
- a leading `NAME=value` environment assignment;
- the `--` and `-args` cutoff, after which a `./` field is a program argument;
- a `cd` or `-C` to an absolute or out-of-root path ending the scan, which is the property that keeps a blocking check from refusing a command it cannot place.

A regression in any of them either misses a refusal or, worse, blocks a valid plan.

**Fix:** Add rows to the existing `TestValidate_VerifyNestedModule` table, not new top-level tests: `go -C=nested test ./pkg` silent, `cd nested; go test ./pkg` silent, `GOFLAGS=-count=1 go test ./nested/pkg` fires, `go test ./pkg -args ./nested/x` silent, and `cd /abs && go test ./nested/pkg` silent.

### [LOW] `importsOf` import-name rules have no test row

**Location:** `internal/planglyph/callercoverage.go:407` (`importsOf`), used by `scanFileReferences`.

**Issue:** An import's name is its alias, else the subject package's clause, else the last path element with a `/vN` suffix dropped, and a blank import binds nothing.
`TestGlyphChain_CallerCoverage` covers the aliased and dot importers and the `other.Method()` exclusion.
It has no row for the major-version suffix drop, for a subject package whose clause differs from its directory name (the `subjectClauses` branch), or for a blank import.
A wrong name in the first two makes a package-level reference unmatched or a method call wrongly excluded.

**Fix:** Add rows to the existing `TestGlyphChain_CallerCoverage` table: an importer of a `.../v2` path calling a deleted member of a package named for the last element before the suffix, and an importer of a package whose clause differs from its directory name, each expected blocking.

## Focus settled

- **Card 5, types load.**
  Bounded: a `context.WithTimeout` on the loader's `timeout` field, with `ctx.Err()` checked after a successful return.
  Logged: the spawn and its completion or failure go through `internal/logger`.
  Offline: `GOPROXY=off` and `GOFLAGS=-mod=readonly` are appended to the environment.
  Every failure path I found falls back to the scan except the partial-AST case in F1: a loader error, a skipped load, a package with no `TypesInfo`, a file built for another GOOS, and an identifier with no object are all answered by the scan.
- **Card 5, `TestMain` and `defaultTypesLoader`.**
  No untagged test can spawn `go list`: `TestMain` replaces `defaultTypesLoader` before `m.Run`, and only the integration file names `goListLoader`.
  No other package's untagged test runs a plan gate over a tree holding a `go.mod`; the skip for a root without `go.mod` covers the rest.
  The timeout is the `goListLoader.timeout` field, not a package-level duration.
- **Card 4.**
  `importPathOf` reads the module path through `modfile.ModulePath` and takes the module from `ModuleOf` over `NestedModules(plan, 0, root)`.
  Name-only method matches collapse into one informational finding per subject, and `uncoveredFinding` is always blocking.
  Not a defect.
- **Cards 1 to 3.**
  `cardGateCommand` and `checkVerifyNestedModule` derive nested modules only through `NestedModules` and `ModuleOf`.
  The `go -C <module>` form passes, and `go test ./nested/...` from the root is refused; both are pinned by test rows and by the new `VerifyNestedModule` parity row.
  Not a defect.
- **Per-card mechanical check.**
  Card 4 holds the only `Delete` group in the plan; `isReference` has no remaining reference anywhere in the repository, and the build is green.
  No card holds a `Rename` group.
- **Test economy.**
  The three new top-level tests (`TestNestedModules`, `TestValidate_VerifyNestedModule`, `TestGlyphChain_CallerCoverageTyped`) each cover a path no existing test covers; the typed rows and the scan rows exercise different resolution paths.
  The only gaps are F2 and F3.
- **Card 7 and the test budget.**
  The `planparser` row rises by two and the `planglyph` row by one, matching the new top-level tests, and the three cards that add them list `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
  `websterengine`, `loomcli` and `contracts/stencils` gain only rows or subtests, so their rows stay.
- **Docs landing.**
  The spec rows, `internal/planglyph/doc.go`, `internal/websterengine/doc.go`, the `validate.go` file comment and `CLAUDE.md` land in the same commits as their code.
  No module table or execution stack changed, so `docs/overview.md` needs no edit, and the plan names no new `PATTERN.md` invariant.
