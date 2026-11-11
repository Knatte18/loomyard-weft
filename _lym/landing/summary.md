# testcover: select an importing package's tests from its coverage map

`internal/testcover` gains a `Selector` that decides, for one importing package, which of its tests a change can affect.
It reads the package's coverage map, compares every linked package the map records with the working tree,
and returns either the selected test names or a `Miss` naming why the map cannot speak for the change.
A miss means the caller runs the importing package whole.
Nothing calls the selector yet; the gate-scope task that consumes it calls `Select` once per importing package.

Why: tiers 0 and 1 carry most of the verify time, and a package whose linked code changed in one function need not rerun every test.
Misses only widen the run, never narrow it.

## What changes

- `Selector` (`NewSelector`, `Select`) in `testselect.go` and `declcompare.go`.
  It takes the working tree, the module's standard-library root and `fabricengine.Fabric.CodeFileAt` as the reader of file content at the map's `Commit`.
- A linked package's non-test Go file counts as changed when its working-tree hash differs from the map's record, or it exists on one side only.
  The selector parses the file at the map's `Commit` and in the working tree with `go/parser`
  and compares each function's receiver, signature and body, ignoring comments and formatting.
  Functions sharing a key name, such as several `init` functions, change together.
  A deleted file makes every function it held changed.
- A test is selected when the map records a covered block of a changed file whose enclosing function changed.
  A test the spawn scan marks (out of process or unresolved) has no recorded coverage,
  so it is selected whenever any linked function changed, and on no other account.
- Misses: the map is missing, unreadable, failed or unmarked, its marker's `Commit` and `BuiltAt` differ from the map's,
  its `Commit` is unreadable, or the package's own files, tests included, differ from the record.
  For a changed linked file the misses are:
  a declaration change outside functions,
  an import removed or added other than a linked or standard-library package,
  a `//go:` line added, removed or changed, a changed method set, an added `init`,
  an added top-level name that is a predeclared identifier, and a file that does not parse.
  A non-Go file that differs misses unless `inert_paths` matches it and no `go:embed` pattern names it.
  The package's own test files count as embed sources.
- Not a miss: comment or formatting edits, a reordered import block,
  and added declarations that no unchanged code can reach (a const outside an `iota` or implicit-value group, a var with no initializer, a type with no methods).
- `NestedPackageDirs` moves from `internal/gatecli` into `internal/testcover` as an exported function,
  so the selector and the testmap tree reader share one definition of which files a package owns.
  `internal/gatecli/testmaptree.go` calls the moved function; behaviour is unchanged.
- `internal/testcover/doc.go` and `docs/overview.md` describe the selector.

## Design decisions to review

- Function-level matching selects a superset of what block-level matching would, and needs no block boundaries from a coverage build.
  The block keys already carry their enclosing function.
- Comparing from the map's own `Commit` lets a map built before or after the tier's base select on changes inside function declarations.
  A change on main since the build counts as a change, which can only add tests.
- Bound: a test that reaches a linked function it did not reach when the map was built, through code unchanged since, is not selected.
  Neither is a test relying on the `init` of a standard-library package that a changed file newly imports.
  Only the landing gate catches these.
- A dependency-drift miss lasts until testmap rebuilds the map by its present rules; this change leaves testmap's staleness rule as it is.
  Coverage narrows only while a map is fresh, and testmap-upkeep owns that cadence.
- A checkout whose `core.autocrlf` converts line endings rehashes every package's own files, so every importing package misses.
- The selector caches file hashes, owned files and file comparisons, so it assumes the working tree does not change while it is in use,
  and it is not safe for concurrent use.

## To check by hand

- `lym loom fmt` and the `internal/testcover` and `internal/gatecli` test suites were not run for this description.
  Run `lym gate test ./internal/testcover ./internal/gatecli` before merging.
