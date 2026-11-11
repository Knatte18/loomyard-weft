# Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by

`selector-non-body-miss` · Note · enhancement, testkit, gate

## Seen (D13 of `testmap-upkeep-design`)

The selector runs a package whole when a change is not a function body (a non-body miss).
The design asks for a note that proposes narrowing this miss.

## Built so far (board tidy, 2026-11-11)

The selector landed as `65416136` (task `coverage-test-select`): `Selector` in `internal/testcover`, with `MissNonBody` the miss reason for "change outside function declarations".
Task `gate-scope-coverage` (`cdc3522a`) now consumes it: the gate scope `coverage` runs, for a package in the diff set only because it imports a changed package, the tests the selector selects, and the whole package on any miss.
The scope report names such a package `miss: <reason>`, so the non-body misses a real gate sees can be counted there.
It already narrows part of the non-body case: comment and formatting edits, a reordered import block,
and added declarations no unchanged code can reach (a const outside an `iota` or implicit-value group, a var with no initializer, a type with no methods) are not misses.
What still misses is the rest: a changed or removed declaration outside functions, an import added or removed other than a linked or standard-library package,
a `//go:` line change, a changed method set, an added `init`, an added predeclared name and a file that does not parse.

Commit `46c3f64b` (task `testmap-selector-upkeep-rules`) changed the selector's other rules, not this miss:
`NewSelector` takes a `Target` (the working tree or a commit);
a package's own non-test Go files are compared by function like a linked package's (`MissOwnFiles` is gone);
a package's own test files are compared per top-level `Test` function, and any other change there is `MissTestFile`
(a test file's import change is no miss when the test binary already links the import);
a changed non-Go file under the package is a miss unless `inert_paths` exempts it;
and a map whose `EnvironmentStamp` differs from the current one is `MissEnvironment`.
The non-body rules above apply to a package's own files as well as to a linked package's.

## Idea

Follow the names a changed declaration is referenced by, and select the tests that reach those references.
Measure with the miss counts that upkeep reports (D12) whether the narrowing pays before building it.
