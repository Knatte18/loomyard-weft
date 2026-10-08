# Webster round 1 — fixer report

All three findings are fixed, each in its own commit.
After the last commit, `go build ./...`, `go vet` under the untagged, `integration`, `tmux` and `llm` tags, `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments` are all green.

## Fixed

### F1 — a file with a parse error was `checked` from its partial syntax

Commit `1a4d97457`.
I confirmed the premise in `golang.org/x/tools@v0.42.0/go/packages/packages.go`: `loadPackage` keeps the partial ASTs from `parseFiles`, records each `scanner.ErrorList` entry as a `ParseError` whose `Pos` begins with the file name, and still type-checks the package.
The new row went RED first: the reference after the parse break was dropped.
- `internal/planglyph/typesload.go`: `typedReferences` no longer walks a file that `pkg.Errors` names in a `ParseError`, so the file stays out of `checked` and the scan decides it whole. The new helper `hasParseError` does the match, and `typedReferences`' doc comment states the rule.
- `internal/planglyph/glyphchain_integration_test.go`: a new `TestGlyphChain_CallerCoverageTyped` row, "a file that does not parse is answered by the scan". A `callees.Target()` call sits after a bad operand, and the row expects it to be blocking.
- `contracts/specs/loom-plan-spec.md`: the `caller-uncovered` row's list of files the load does not check now includes a file that does not parse.

### F2 — `verify-nested-module` branches with no test row

Commit `e53cbe5a2`.
New rows in the existing `TestValidate_VerifyNestedModule` table, with no new top-level test.
Each row's verdict differs from what it would be if its branch were missing:
- `go -C=nested test ../other/pkg` fires. It would be silent if `-C=` were not parsed, because the scan would end.
- `cd nested; go test ../other/pkg` fires (`;` separator).
- `cd nested` and `go test ../other/pkg` on two lines of a card `Verify:` value fires (newline separator).
- `GOFLAGS=-count=1 go test ./nested/pkg` fires (leading env assignment).
- `go test ./pkg -args ./nested/x` is silent (`-args` cutoff).
- `cd /abs && go test ./nested/pkg` is silent (absolute `cd` ends the scan).
- `go -C .. test ./nested/pkg` is silent (`-C` out of the root ends the scan).

### F3 — `importsOf` import-name rules with no test row

Commit `51dd27b0f`.
New rows in the existing `TestGlyphChain_CallerCoverage` table, and both new files are added to `coverageFiles` so a wrong match would show up in the finding:
- "an import binds its last path element without a major-version suffix": `import "example.com/other/v2"` then `other.Method()`. The row expects only the fixture's own informational `callers/callers.go`. If `/v2` were not dropped, the import would bind `v2`, and `usev2.go` would join the informational finding.
- "an import of a subject's package binds its package clause": deleting `dirname#DirDiffers` (package `clausename`) with an unaliased importer calling `clausename.DirDiffers()` is blocking.

## Disputed

- F3, blank-import part only: the finding lists the blank import as uncovered.
  In `importsOf`, the `_` case only keeps `_` out of the `imports` map.
  `_` can never be a selector qualifier in Go, so an `imports["_"]` entry could never match a site.
  No test can observe this branch.
  The finding's own **Fix:** line asks for only the two rows above, and both are added.

## Deferred

None.
