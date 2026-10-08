# Card 5 — caller-scan-types

**Create:**
- `internal/planglyph#typesLoader`
- `internal/planglyph#goListLoader`
- `internal/planglyph#goListLoader.load`
- `internal/planglyph#defaultTypesLoader`
- `internal/planglyph#candidateFiles`
- `internal/planglyph#typedReferences`
- `internal/planglyph#isSubjectObject`
- `internal/planglyph/typesload.go`

**Edit:**
- `internal/planglyph#planGatePass` -> `func planGatePass(plan *planparser.Plan, worktreeRoot string, loader typesLoader) ([]Finding, error)`
- `internal/planglyph#ValidateFormat`
- `internal/planglyph#ValidateFormatAfter`
- `internal/planglyph#Validate`
- `internal/planglyph#callerCoverageFindings` -> `func callerCoverageFindings(plan *planparser.Plan, lang glyph.Language, worktreeRoot string, answers map[string]quarry.ResolveResult, loader typesLoader) ([]Finding, error)`
- `internal/planglyph#scanReferences` -> `func scanReferences(root string, files []string, subjects []coverageSubject) ([]map[string][]int, error)`
- `internal/planglyph#coverageSubject`
- `internal/planglyph#coverageSubjects`
- `internal/planglyph/testmain_test.go`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`
- `go.mod`
- `go.sum`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#NestedModules`
- `internal/planparser#ModuleOf`

**Intent:** Put type information in front of card 4's scan, so `caller-uncovered` tells receivers apart (D1) under a bounded, logged, offline load that degrades to the scan instead of erroring (D2); this closes both false-positive classes of issue #461.

The module gains `golang.org/x/tools` as a direct requirement in `go.mod` and `go.sum`.

`typesLoader` is the seam: an interface whose one method `load(root string, dirs []string) ([]*packages.Package, error)` loads the package directories `dirs`, worktree-relative, under `root`.
`goListLoader` is the real one, with a `timeout time.Duration` field; `load` calls `packages.Load` with `Dir` the told root, `Env` the process environment plus `GOPROXY=off` and `GOFLAGS=-mod=readonly`, a context under the timeout, `Tests: true`, `BuildFlags` `-tags=integration,tmux,llm`, and a mode of name, files, compiled files, imports, types, syntax and types info without deps, so imports come from compiler export data.
It logs the spawn before the load and its completion or failure after it through `internal/logger` (`PATTERN-spawn-observability`).
`defaultTypesLoader` is `goListLoader{timeout: 3 * time.Minute}`; the exported `ValidateFormat`, `ValidateFormatAfter` and `Validate` pass it to `planGatePass`, which passes it on to `callerCoverageFindings`.

`callerCoverageFindings` now runs in four steps.
`candidateFiles` is today's walk with its directory skips, tokenizing each file and returning the files holding an identifier named like any subject, without deciding a match.
The load covers the directories of those candidates plus each subject's declaring directory, minus every directory whose `ModuleOf(NestedModules(plan, 0, worktreeRoot), dir)` is not `.`, since the load runs from the root module (D5); with no directory left it is skipped.
It is also skipped, with no loader called, when the told worktree root holds no `go.mod`: a root-module load cannot succeed there, so D2's fallback answers it without a spawn.
`typedReferences` walks every syntax file of every loaded package: a file whose package carries types info is `checked`, and in it an identifier whose `TypesInfo.Uses` object satisfies `isSubjectObject` is a reference at its line, deduplicated by file and line across a package and its test variant.
An identifier in a checked file named like a subject that has neither a `Uses` nor a `Defs` object is recorded in `unresolved`, indexed like `subjects`, as its file and line.
`isSubjectObject` matches a package-level subject by declaring package path (the subject's `importPath`), name and package-scope parent, and a method subject by declaring package path, receiver base type name (through `types.Func.Origin` and the named type's `Origin` for a generic receiver; `coverageSubject` gains the receiver type name from the glyph's owner) and method name, so a promoted call through an embedding struct and a method value `t.M` count, and a call on another type or through an interface does not.
Card 4's scan, re-signed `scanReferences` taking the file list, runs over every candidate file not in `checked` plus every checked file holding an `unresolved` line.
`callerCoverageFindings` then combines the two per subject and file.
It drops a checked file's scan matches at every line `unresolved` does not record for that subject, so a typed non-reference such as `u.Method()` on another type never returns as a name match.
For a package-level subject, the typed lines and the kept scan lines of one file merge, sorted and deduplicated, into the one blocking `uncoveredFinding` for that subject and file, since both are resolved.
For a method subject, a kept scan line that `typedReferences` already counts for that subject and file is dropped, so a typed reference is never counted twice.
When the load fails or times out, `checked` is empty and every candidate goes through the scan; a load failure is never `ErrQuarryUnavailable` and returns no error.
A typed reference is resolved, so a method's typed references are blocking `uncoveredFinding`s per subject and file like a package-level member's; a method's remaining scan matches stay the one collapsed informational finding.
The declaration-span exclusion, coverage and admissibility stay as they are.

Bound, stated in the spec row and the doc: a call through an interface whose method set holds the method is no longer reported, where it was an informational name match before; a re-signed method that stops satisfying an interface breaks compilation at assignment sites this check does not see, which the card gate's root build catches in untagged root-module code, its integration step only in the card's own directories, the plan-level verify in tagged files, and no gate for a file built only for another GOOS or in a nested module unless a command runs inside it.
Without type information the check is never weaker than card 4's.

The untagged tier spawns nothing (`PATTERN-test-speed`): `testmain_test.go`'s `TestMain` sets `defaultTypesLoader` to a test loader that returns an error without spawning, before `m.Run`, with a comment naming that process-global state; so every untagged test in the package runs the "load failed outright" path, and `TestGlyphChain_CallerCoverage`'s header says its rows pin that the gate then answers with the scan's findings and no error.
Another package's untagged test cannot replace `defaultTypesLoader`, and the no-`go.mod` skip is what keeps it from spawning: `internal/webstercli`'s `TestValidateCmd_Envelopes` runs `planglyph.Validate` over `deleteOrderPlan`, which deletes `sub#Foo` that `deleteOrderFiles`' `sub/b.go` calls, in a tree with no `go.mod`.
No other package's untagged test runs a plan gate over a delete or re-sign in a tree holding a `go.mod`.

`TestGlyphChain_CallerCoverageTyped`, new in `glyphchain_integration_test.go`, calls `planGatePass` with `goListLoader{timeout: time.Minute}` over a fixture copy, one table whose rows write their extra files:
a same-named method `Method` on another type, called `u.Method()`, not reported for `callees#Thing.Method`;
a package-qualified `pkg.Method` naming a function of another package, not reported;
a call promoted through an embedding struct and a method value `t.Method`, each blocking;
a call through an interface whose method set holds `Method`, not reported;
an aliased and a dot importer of `callees#Target`, each blocking through the typed path;
a file constrained to another GOOS (`//go:build plan9`) calling `callees.Target()` and `t.Method()`, the first blocking and the second one collapsed informational finding saying the receiver was unresolved;
a nested module beside those root callers, which leaves the typed verdicts of the root files intact;
a type-checked file holding `undefined().Method()`, whose `Method` has no object since its receiver's type failed, alone on one line, the same call beside a typed `t.Method()` on a second line, and a typed `u.Method()` on a third: the first line reported through the scan in the collapsed informational finding, the second blocking and absent from that finding, the third not reported;
and a fixture copy with its root `go.mod` removed, whose loader fails the test when called, answered with the scan's findings.
The new top-level test raises `internal/planglyph`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

Docs in the same commit: the spec's `caller-uncovered` row and the `Edit`/`Delete` lines of its card-types table, `internal/planglyph/doc.go`'s bullet (blocking for a resolved reference, method or package-level, informational and collapsed for a name-only method match) and `callercoverage.go`'s file comment describe the typed load, its skip for a root without `go.mod`, its fallback, the bound above, and card 4's shadowed-import bound narrowed to the fallback.

**ImpactSummary:** `planGatePass`, `callerCoverageFindings` and `scanReferences` re-signed inside `internal/planglyph`, every caller on this card; the exported entry points keep their signatures, and type-resolved method callers become blocking at both plan gates.

**Commit:** 5: caller-uncovered resolves references with go/types and falls back to the import-path scan
