# Card 4 — caller-scan-import-path

**Create:**
- `plan:internal/planglyph#importPathOf` -> `func importPathOf(worktreeRoot string, modules []string, dir string) (string, error)`
- `plan:internal/planglyph#referenceSite` -> `type referenceSite struct`
- `plan:internal/planglyph#referencesSubject` -> `func referencesSubject(subject coverageSubject, site referenceSite) bool`
- `plan:internal/planglyph#unresolvedMethodFinding` -> `func unresolvedMethodFinding(subject coverageSubject, references map[string][]int) Finding`

**Delete:**
- `internal/planglyph#isReference`

**Edit:**
- `internal/planglyph#coverageSubject`
- `internal/planglyph#coverageSubjects`
- `internal/planglyph#scanReferences`
- `internal/planglyph#scanFileReferences`
- `internal/planglyph#callerCoverageFindings`
- `internal/planglyph#uncoveredFinding`
- `internal/planglyph/glyphchain_test.go`
- `go.mod`
- `go.sum`

**Prosa:**
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#NestedModules`
- `internal/planparser#ModuleOf`

**Intent:** Rewrite `caller-uncovered`'s name scan into the import-path resolution of D3, with D4's severity split and D5's nested-module reach; this scan is the whole reference walk until card 5 puts type information in front of it, and stays its fallback afterwards.

`importPathOf` answers a package directory's import path: the module path of the `go.mod` in `ModuleOf(modules, dir)` (the root's own `go.mod` for `.`), read with `golang.org/x/mod/modfile`'s `ModulePath`, joined with `dir` relative to that module; it returns "" when that `go.mod` does not exist, and an error only when one exists but cannot be read.
`go.mod` and `go.sum` gain `golang.org/x/mod` as a direct requirement (Shared Decisions).
`coverageSubjects` reads `NestedModules(plan, 0, worktreeRoot)` once, the on-disk set, and stores on each package-level `coverageSubject` its new `importPath` field beside `dir` and `packageClause`; a read error stays wrapped in `ErrQuarryUnavailable`, as the package-clause read is today.

`scanFileReferences` keeps tokenizing (comments and string literals still never match) and additionally reads the file's import block, through `go/parser` with `parser.ImportsOnly`, into a name-to-path map and a dot-import set.
An import's name is its alias, or else the default name: the subject's own `packageClause` when the path is a subject's `importPath`, and otherwise the path's last element with a `/vN` major-version suffix dropped.
For every identifier named like a subject it builds a `referenceSite` (the file's directory and package clause, whether a `.` precedes the identifier, the identifier left of that `.`, and the file's imports) and asks `referencesSubject`, which replaces `isReference`:
- a package-level subject is referenced inside its declaring package (same directory, same package clause) by its bare identifier not after `.`;
  elsewhere, by an import of the subject's `importPath` under that import's name followed by `.` and the identifier, or by the bare identifier when that import is a dot import;
  with an empty `importPath` it keeps today's package-clause rule;
- a method subject is referenced by `.` and its identifier, except when the identifier left of the `.` is a name the file imports, which drops the `batcher.Batch` class of issue #461 even without types.
The declaration-span exclusion stays as it is.

Bound, stated in the spec row: the import-name exclusion also drops a method call on a local variable or parameter that shadows an imported package's name (`errors.M()` after `errors := …`), which is then not matched even informationally.
It is bounded to the scan: until card 5 lands that is every file, and afterwards only a file or identifier the load left without type information, since a type-checked call resolves through its receiver's type.
The walk keeps today's file set (D5): it skips only `testdata`, `vendor`, `.`- and `_`-prefixed directories, so a nested module's files are scanned, and a file there that imports a root package by its import path references a root subject by the same rule; a subject declared in a nested module gets its `importPath` from that module's `go.mod`.

`callerCoverageFindings` keeps coverage and admissibility as they are, then reports by resolution (D4): a package-level subject's uncovered references stay one blocking `uncoveredFinding` per subject and file, and `uncoveredFinding` is now always blocking, since every reference it reports is resolved.
A method subject's uncovered name matches collapse into one informational `unresolvedMethodFinding` per subject, naming every file with its lines and stating that the receiver could not be resolved; its way forward is the same as `uncoveredFinding`'s.
Card 5 makes a type-resolved method reference blocking; until then every method match is name-only.

Docs in the same commit: the `caller-uncovered` row of the spec's validation-checks section is rewritten for the import-path rule, the severity split, the collapse and the shadowed-import bound above, and its old bound about aliased and dot imports is dropped, keeping the bound for a caller in a file an earlier card creates; `internal/planglyph/doc.go`'s `caller-uncovered` bullet and `callercoverage.go`'s file comment say the same, and `scanReferences`' doc comment replaces its package-clause and method rules with the import-path rule and the import-name exclusion above.

Tests: `TestGlyphChain_CallerCoverage` stays untagged and gains rows through its `files` map, each extra file named in `coverageFiles` so the detail parsing finds it:
an aliased importer calling `c.Target()` and a dot importer calling `Target()`, each blocking;
a deleted `callees#Thing.Method` with name matches in two files, reported as one informational finding naming both files;
a selector `other.Method()` on an imported package that declares a function `Method`, which is no match;
a nested module (`nested/go.mod`, a `replace` of the fixture module to `../`) whose file calls `callees.Target()`, blocking, and `t.Method()`, folded into the informational finding;
and a subject declared in that nested module with a caller importing it by the nested module's path, blocking.
The existing "delete of a method" row moves to the collapsed shape.
No new top-level test, so `internal/planglyph`'s budget row is unchanged.

**ImpactSummary:** internal to `caller-uncovered` in `internal/planglyph`; package-level findings gain the aliased and dot-import callers, method findings collapse to one per subject, and no exported signature changes.

**Commit:** 4: caller-uncovered resolves callers by import path and collapses name-only method matches
