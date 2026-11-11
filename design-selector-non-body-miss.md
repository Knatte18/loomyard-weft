# Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by

`selector-non-body-miss` · Note · enhancement, testkit, gate

## Seen (D13 of `testmap-upkeep-design`)

The selector runs a package whole when a change is not a function body (a non-body miss).
The design asks for a note that proposes narrowing this miss.

## Idea

Follow the names a changed declaration is referenced by, and select the tests that reach those references.
Measure with the miss counts that upkeep reports (D12) whether the narrowing pays before building it.
