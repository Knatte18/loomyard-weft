# Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by

`selector-non-body-miss` · Note · enhancement, testkit, gate

## Seen (D13 of `testmap-upkeep-design`)

The selector runs a package whole when a change is not a function body (a non-body miss).
The design asks for a note that proposes narrowing this miss.

## Built so far (board tidy, 2026-11-11)

The selector landed as `65416136` (task `coverage-test-select`): `Selector` in `internal/testcover`, with `MissNonBody` the miss reason for "change outside function declarations".
Nothing calls it yet; task `gate-scope-coverage` consumes it.
It already narrows part of the non-body case: comment and formatting edits, a reordered import block,
and added declarations no unchanged code can reach (a const outside an `iota` or implicit-value group, a var with no initializer, a type with no methods) are not misses.
What still misses is the rest: a changed or removed declaration outside functions, an import added or removed other than a linked or standard-library package,
a `//go:` line change, a changed method set, an added `init`, an added predeclared name and a file that does not parse.

## Idea

Follow the names a changed declaration is referenced by, and select the tests that reach those references.
Measure with the miss counts that upkeep reports (D12) whether the narrowing pays before building it.
